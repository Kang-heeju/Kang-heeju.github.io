---
title: "[Typescript] WebSocket 사용해보기(Quant Bot)"
last_modified_date: "2024-10-31 22:53:38"
parent: "TypeScript & NestJS"
---

<div style="font-size:32px; font-weight: 800; border-left: 7px solid #0687f0; padding-left:15px !important; color:#000000; margin-bottom:15px;">[Typescript] WebSocket 사용해보기(Quant Bot)</div>

{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

### 도입하게 된 계기

- 아비트리지 봇을 개발하면서, 실시간성에 관한 문제를 맞닥뜨리게 되었다. 

  - 아비트리지 봇은 서로 다른 거래소 간(보통 CEX/DEX 간의) 가격 차이를 이용하여 무위험 수익을 얻기 위해 설계된 자동화 트레이딩 시스템이다. 특정 자산이 한 거래소에서는 낮은 가격에 거래되고 있고, 한 거래소에서는 높은 가격에 거래되고 있을 때 두 시장에서 동시에 매수&매도 (long&short)함으로서 차익을 얻는 전략이다. 
  - 이러한 아비트리지 봇은 계속 변동하는 코인 가격의 시세를 정확히 파악하여 스프레드가 발생했을 때 정확한 gap을 판단하고 시장에 진입하는 것이 중요하다. 
  - 기본적으로 투자 봇은 RESTful API를 활용한다. 하지만 실시간성이 매우 중요한 아비트리지 봇 특성상, 서버 자원 보호를 위해 호출 제한이 걸려 있는 REST API는 아비트리지 알고리즘에 적합하지 않았다. 
  - 실제로 거래소 간 스프레드는 매우 짧은 시간 동안만 벌어지기 때문에, 해당 갭을 정확히 캐치하지 못한다면 잘못된 스프레드에 진입하여 손실을 볼 수 있다. 
  - 처음에는 RESTful API를 활용해서 투자 알고리즘을 짰었는데, 딜레마에 빠지게 되었다. 실시간성을 높이기 위해 호출을 너무 자주 하게 된다면 request limit에 걸려 서버가 터져버리거나, 호출을 적게 하게 되면 정확한 갭을 판단하지 못하고 잘못된 판단을 하게 되는 경우가 발생하였다. 

  ![arbitrage](../../../assets/images/typescript/rdb-arbitrage.png)

-> 실제로 이 사진을 보면, 내가 진입하기 위해 설정한 갭은 보통 0.3 ~ 0.5%지만, openGap을 보게 되면 제대로 들어가는 케이스는 거의 없는 것을 확인할 수 있다. 모두 REST API의 한계로 발생한 문제이다. 



- 분석 결과 판단한 문제는 2가지 정도였다. 

  - 실시간 가격을 트래킹 해 걸어 놓은 지정가 주문을 수정해야 하지만 REST API 활용시 정밀한 지정가 주문 수정의 어려움

    - 따라서 잘못된 갭을 캐치해서 포지션 진입하는 케이스가 발생

  - 아비트리징 기회를 탐색하기 어려움

    - 이미 시장에 너무 많은 아비트리저들이 존재하기 때문에, 정확히 스프레드를 캐치하기 위해서는 오차 시간 0.1초 미만의 실시간성이 중요함

      

- REST API는 각 요청에 대한 독립적인 HTTP 연결을 사용하므로 다수의 요청을 처리할 때 과부하가 생길 수 있지만, websocket은 하나의 socket을 통해 지속적인 양방향 통신이 가능하기 때문에 실시간 데이터를 효율적으로 주고받을 수 있다.  

- 따라서 아비트리지 전략에는 RESTful보다 websocket이 훨씬 효율적이라고 판단했고, 알고리즘을 websocket에 맞춰 수정하기로 했다. 



### WebSocket?

![Websocket](../../../assets/images/typescript/websocket/websocket.png)

WebSocket은 웹 앱 서버 간의 지속적인 연결을 제공하는 프로토콜이다. 이를 통해 서버와 클라이언트의 양방향 통신이 가능해진다. 기존 HTTP 통신 방법은 request에 따른 response만 가능하다고 한다면, websocket 연결은 한 번 열린 후 계속 유지되므로 서버나 클라이언트에서 언제든지 데이터 전송이 가능하다. 그렇기에 **실시간으로 진행하는 통신에서 적극적으로 사용한다.**

Websocket은 HTML5 등장 실시간 웹 어플리케이션을 위해 설계된 통신 프로토콜이며, TCP를 기반으로 하기 때문에 신뢰성 있는 데이터 전송을 보장하며 순서가 보장된 양방향 통신을 제공한다. HTTP와 다르게 클라이언트와 서버 간에 최초 연결이 이루어지면, 이 연결을 통해 양방향 통신을 지속적으로 할 수 있다.

이때 데이터는 **패킷(packet)** 형태로 전달되며, 전송은 연결 중단과 추가 HTTP 요청 없이 양방향으로 이뤄진다.



## WebSocket 연결 과정

Typescript의 'ws'라이브러리를 사용했다.  ws 라이브러리는 Node.js 환경에서 WebSocket 통신을 간편하게 구현할 수 있는 라이브러리이다. 

거래소에서는 보통 사용할 수 있는 websocket 서버 주소를 제공해 주기 때문에, 해당 서버에 연결하여 데이터를 주고받을 수 있다. 

또한 거래소 별로 제공하는 정보에 따라 private Websocket 주소, public WebSocket 주소가 따로 존재하기 때문에 2개의 연결을 뚫어야 했다. 

```typescript
import 'dotenv/config';
import WebSocket from 'ws';
import crypto from 'crypto';
import { v4 as uuidv4 } from 'uuid';
import { AxiosHttpClient } from '../network/axios/AxiosHttpClient.js';
import { WooRequestHandler } from '../exchange/woo/WooRequestHandler.js';
import { WooClient } from '../exchange/woo/WooApi.js';

export class WooPrivateStream {
  private orderId: string;
  private wooAccountStream: WebSocket;
  private wooApi: WooClient;

  constructor() {
    const client = new AxiosHttpClient();
    const wooRequestHandler = new WooRequestHandler(client);
    this.wooApi = new WooClient(wooRequestHandler);

    this.wooAccountStream = new WebSocket(
      `wss://wss.woo.org/v2/ws/private/stream/${process.env.WOO_APPLICATION_ID}`,
    );

    this.wooAccountStream.on('open', this.onOpen.bind(this));
    this.wooAccountStream.on('message', this.onMessage.bind(this));
    this.wooAccountStream.on('error', this.onError.bind(this));
    this.wooAccountStream.on('close', this.onClose.bind(this));
  }
```

하나의 웹소켓 별로 하나의 클래스로 모듈화하여 개발을 진행했다. 거래소 2곳의 public, private WebSocket이 필요했기에 총 4개의 클래스를 설계했다.

생성자로 websocket 인스턴스와 HTTP 인스턴스를 생성해서 websocket으로 받아오는 정보들로 필요 시 REST API를 함께 활용할 수 있도록 했다. 

또한 이벤트 핸들러를 이용해 각 이벤트가 발생했을 때 처리할 함수들을 바인딩해줬다. 

### onOpen()

```typescript
  private onOpen() {
    console.log('[WooX] starting Woox socket..');
    tradingSymbols.forEach((symbol) => {
      const orderbookSubscribeMessage = {
        id: uuidv4().substring(0, 32),
        topic: `${symbol.wooxSymbol}@orderbookupdate`,
        event: 'subscribe',
      };
      this.wooAccountStream.send(JSON.stringify(orderbookSubscribeMessage));
    });
  }

```

연결이 성공적으로 열렸을 때, 필요한 구독들을 공식 tech docs를 참고해 구독 요청하도록 설계하였다.



### onMessage()

- 다음은 웹소켓 연결에 성공했을 시 응답으로 오는 packet의 예시이다. 
  - topic으로 어떤 구독에 대한 정보인지 넘어오고, ts로 정보가 전달된 시간을 linux timestamp로 보여주며 data에 원하는 정보가 담겨서 오게 된다.


<img src="../../../assets/images/typescript/websocket/woox-websocket.png" alt="woox-websocket" width="500"/>

- 해당 코드는 실제로 내가 아비트리지 봇에 사용했던 구독 message 핸들링 코드의 일부이다. 

```typescript
  private async onMessage(data: WebSocket.Data) {
    if (!isWoosocketRunning) {
      this.wooAccountStream.close();
    }
    const parsedData = JSON.parse(data.toString());
    if (parsedData.event && parsedData.event === 'ping') {
      this.wooAccountStream.send(
        JSON.stringify({
          event: 'pong',
          ts: Date.now(),
        }),
      );
    }
    if (parsedData.topic && parsedData.topic.includes('orderbookupdate')) {
      await this.handleData(parsedData.data);
    }
  }

  private async handleData(data: any) {
    const { asks, bids, symbol } = data;
    const wooxSymbol = symbol;
    const index = tradingSymbols.findIndex(
      (item) => item.wooxSymbol === wooxSymbol,
    );
    //현재 이미 수정중인지 확인
    if (tradingSymbols[index].isEditing) {
      console.log('Already editing..', wooxSymbol);
      return;
    }
    const ask = asks[0];
    const bid = bids[0];
    if (ask === undefined || bid === undefined) {
      return;
    }
    const { buyPrice, sellPrice } = this.calcPrice(ask[0], bid[0], wooxSymbol);
    const orderlySymbol = wooxSymbol.replace('USDT', 'USDC');
    const orderSize = this.getOrderSizeByWooxSymbol(wooxSymbol);
    const opengap = this.findOpenGapByWooxSymbol(wooxSymbol);

    try {
      if (orderSize === undefined) {
        console.error(
          '[WEBSOCKET] error in woox websocket: orderSize is undefined',
        );
        return;
      }
      if (!isPosition[index] && isOpenOrder[index]) {
        //포지션이 없고 주문이 있을 때
        //지정가 주문 넣은 가격과 필요한 주문가격의 차이가 0.3% 이상 -> 주문 수정
        if (
          Math.abs(orderlyBuyPrice[index] - bid[0]) / orderlyBuyPrice[index] >
            opengap + 0.003 ||
          Math.abs(orderlySellPrice[index] - ask[0]) / orderlySellPrice[index] >
            opengap + 0.003 ||
          Math.abs(orderlyBuyPrice[index] - bid[0]) / orderlyBuyPrice[index] <
            opengap - 0.003 ||
          Math.abs(orderlySellPrice[index] - ask[0]) / orderlySellPrice[index] <
            opengap - 0.003
        ) {
          //LONG 주문이나 SHORT 주문 수정 필요하면
          tradingSymbols[index].isEditing = true;
          const orderlyOpenOrdersInfos =
            await this.orderlyApi.getOrderlyOpenOrders();

          if (
            Math.abs(orderlyBuyPrice[index] - bid[0]) / orderlyBuyPrice[index] >
              opengap + 0.003 ||
            Math.abs(orderlyBuyPrice[index] - bid[0]) / orderlyBuyPrice[index] <
              opengap - 0.003
          ) {
            //LONG 주문 수정
            if (orderlyOpenOrdersInfos.length !== 0) {
              const currentTime = new Date();
              console.log(
                `[${currentTime.toLocaleString()}]Update Order of LONG..., ${wooxSymbol}`,
              );
              console.log(
                `Order Gap of LONG: ${Math.abs(orderlyBuyPrice[index] - bid[0]) / orderlyBuyPrice[index]}\n`,
              );
              const orderlyOpenOrderInfo = orderlyOpenOrdersInfos.filter(
                (order) => order.symbol === orderlySymbol,
              );
              tradingSymbols[index].wooxBidPrice = bid[0];
              if (
                orderlyOpenOrderInfo[0].side === 'BUY' &&
                isOpenOrder[index]
              ) {
                await this.orderlyApi.editOrderlyOrder(
                  orderlyOpenOrderInfo[0].order_id.toString(),
                  orderlySymbol,
                  buyPrice,
                  'LIMIT',
                  orderlyOpenOrderInfo[0].side,
                  orderSize,
                );
                orderlyBuyPrice[index] = buyPrice;
              } else if (
                orderlyOpenOrderInfo[0].side === 'SELL' &&
                isOpenOrder[index]
              ) {
                await this.orderlyApi.editOrderlyOrder(
                  orderlyOpenOrderInfo[1].order_id.toString(),
                  orderlySymbol,
                  buyPrice,
                  'LIMIT',
                  orderlyOpenOrderInfo[1].side,
                  orderSize,
                );
                orderlyBuyPrice[index] = buyPrice;
              } else {
                console.log('No update in LONG order..', wooxSymbol);
                console.log(orderlyOpenOrderInfo);
              }
            } else {
              console.log(
                '[WEBSOCKET] Cannot get open order infos..',
                wooxSymbol,
              );
            }
          }
          if (
            Math.abs(orderlySellPrice[index] - ask[0]) /
              orderlySellPrice[index] >
              opengap + 0.003 ||
            Math.abs(orderlySellPrice[index] - ask[0]) /
              orderlySellPrice[index] <
              opengap - 0.003
          ) {
            //SHORT 주문 수정
            if (orderlyOpenOrdersInfos.length !== 0) {
              const currentTime = new Date();
              console.log(
                `[${currentTime.toLocaleString()}]Update Order of SHORT..., ${wooxSymbol}`,
              );
              console.log(
                `Order Gap of SHORT: ${Math.abs(orderlySellPrice[index] - ask[0]) / orderlySellPrice[index]}\n`,
              );
              const orderlyOpenOrderInfo = orderlyOpenOrdersInfos.filter(
                (order) => order.symbol === orderlySymbol,
              );
              tradingSymbols[index].wooxAskPrice = ask[0];
              if (
                orderlyOpenOrderInfo[0].side === 'BUY' &&
                isOpenOrder[index]
              ) {
                await this.orderlyApi.editOrderlyOrder(
                  orderlyOpenOrderInfo[1].order_id.toString(),
                  orderlySymbol,
                  sellPrice,
                  'LIMIT',
                  orderlyOpenOrderInfo[1].side,
                  orderSize,
                );
                orderlySellPrice[index] = sellPrice;
              } else if (
                orderlyOpenOrderInfo[0].side === 'SELL' &&
                isOpenOrder[index]
              ) {
                await this.orderlyApi.editOrderlyOrder(
                  orderlyOpenOrderInfo[0].order_id.toString(),
                  orderlySymbol,
                  sellPrice,
                  'LIMIT',
                  orderlyOpenOrderInfo[0].side,
                  orderSize,
                );
                orderlySellPrice[index] = sellPrice;
              } else {
                console.log('No update in SHORT order..', wooxSymbol);
                console.log(orderlyOpenOrderInfo);
              }
            } else {
              console.log(
                '[WEBSOCKET] Cannot get open order infos..',
                wooxSymbol,
              );
            }
          }
        }
      }
    } finally {
      tradingSymbols[index].isEditing = false;
    }
  }
```

해당 코드는 코인 오더북의 실시간 정보를 real-time에 가깝게 받아오는데, 해당 메시지에서 가격 변동에 따라 지정가 주문을 수정하게 되는 전략이다. 



> **다음과 같이 실시간 처리가 필요한 부분은 WebSocket을 활용해 전략을 수정하였고, 주문을 수정하거나 하는 부분은 기존에 사용하던 REST API 방식으로 구현하였다.**
>
> **실제로 주문 지연이 발생하던 케이스였던 가격 변동을 트래킹하는 부분이나, 한 거래소에서 주문 체결된 것을 확인하는 전략을 WebSocket으로 수정하면서 아비트리징 전략이 실행되는 시간을 약 30%가량 줄일 수 있었다.**



### 추가적인 개선 가능 사항

- gRPC를 사용하거나, 직접 node 서버를 구축한다면 더욱 빠른 실시간 통신이 가능할 것
- UDP 기반의  HTTP/3 활용 
  - 하지만 신뢰성의 문제를 해결할 수 있는 방법이 있는지 확인해 볼 필요가 있음
- REST API 대신 GraphQL 사용
  - 필요한 데이터만 보낼 수 있다는 장점이 있으나 거래소 별로 해당 기능을 제공하는 지 확인이 필요하다. 




## REF

- [웹 소켓 (Web Socket)이란?](https://velog.io/@sj_yun/Web-Socket%EC%9D%B4%EB%9E%80)

- [WooX Tech Docs](https://docs.woox.io/#orderbookupdate)
