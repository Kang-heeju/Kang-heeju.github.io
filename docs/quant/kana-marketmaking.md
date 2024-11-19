---
title: "트레이딩 프로젝트-Kana Labs Quant Bot"
last_modified_date: "2024-11-19 21:33:38"
parent: "Quant"
---

<div style="font-size:32px; font-weight: 800; border-left: 7px solid #0687f0; padding-left:15px !important; color:#000000; margin-bottom:15px;">트레이딩 프로젝트-Kana Labs Quant Bot</div>

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

 기존에 네이버 블로그에 작성해둔 글을 테크 블로그를 만들면서 옮겨오려고 한다. 

지난 3개월간 주식이랑 암호화폐 쪽을 좀 열심히 들여다 보면서 나름 많이 성장한 여름방학이 됐던 것 같다.



지난 한달 동안은 트레이딩 전략을 따라하려다가 수익도 보고,, 잃기도 하고(결국 청산 엔딩) 했으나

이번에 좀 의미 있는 수익이 나온 이벤트가 있어서 기록해 두려고 한다.



![kana](../../../assets/images/quant/kana/kana.png)

이벤트는 Kana Labs라는 DeFi 거래소에서 진행했던 리워드 프로그램이다.

[Kana Labs Introduces Exclusive Referral & Rewards Program-medium](https://medium.com/kana-labs/kkana-labs-introduces-exclusive-referral-rewards-program-32dbfb2865b4)

잘 읽어보면 거래량/레퍼럴 포인트에 비례해서 등수별로 Kana 코인과 Aptos 코인을 차등 지급한다고 되어 있는데, 제 눈에 들어왔던건 거래량 대비 Aptos 코인 지급 이벤트였다. 

Kana 코인은 아직 비상장코인이고, 가격이 얼마나 될지도 모르고 레퍼럴 많이 한 사람을 이길수가 없기에..  순전히 거래량만으로 승부를 보는 Aptos 코인을 노렸다. 

리더보드를 보고 조금만 해도 순위권에 들 수 있겠다고 생각해서 참가해 보기로 했다.

![leaderboard](../../../assets/images/quant/kana/leaderboard.png)

해당 사진은 이벤트 마감 약 3-4일 정도 전의 리더보드 사진인데, 거래량이 다른 거래소 이벤트들보다 현저히 낮은 것을 확인할 수 있다.

요즘은 워낙 스캠 거래소도 많아서 거래소도 잘 보고 해야 하는데(실제로 입금은 되지만 출금이 되지 않는 곳도 존재), kana labs는 얼마 전 SKT랑 협업하기도 했었고 나름 Swap 쪽으로는 유명한 거래소 같아서 믿을만한 거래소 같았다.

실제로 소액으로 입금/출금 테스트 결과 제대로 동작했다. 



**보통 이런 거래소 이벤트는 무조건 투자 봇을 돌리는 것이 이득이다. 손매매로 하면 심리가 섞이게 되고, 결국 원칙에 따른 매매를 못하기 때문이다.**

![api](../../../assets/images/quant/kana/api.png)

보통 거래소들은 투자 봇을 위한 tech docs나 REST/Websocket을 제공하는데, kana 거래소는 GitHub 링크 하나를 딱 던져준다.

사실 이거라도 쓰면 문제가 되지 않겠는데...

![api](../../../assets/images/quant/kana/api-key.png)

API Key가 없으면 못 쓰는 것 같다. 

해당 메일로 key 요청을 보냈지만, 회신은 오지 않아 다른 방법을 찾기로 했다. 

그래서 결국 구글링 중 찾게 된 것이 kana의 npm 라이브러리였다.

[@kanalabs/trade-npmjs](https://www.npmjs.com/package/@kanalabs/trade)

해당 라이브러리는 Aptos노드를 써서 직접 노드에 트랜잭션을 보내 이를 블록에 포함시키는 방식으로 동작한다. 

Aptos의 합의 알고리즘의 원리로, 사용자가 노드에 명세가 적힌 트랜잭션을 전송하면, 노드는 이를 블록에 포함하기 위해 여러 validator가 검증을 수행하게 된다. 각 validator는 트랜잭션의 유효성을 확인하고 합의 과정을 통해 블록의 순서와 내용을 결정한다. 합의가 완료되면 블록은 체인에 추가된다.

만약 거래소에서 제공하는 API를 사용한다면 노드와의 상호작용을 간소화하여 트랜잭션 요청을 효율적이고 비교적 빠르게 처리할 수 있지만, 이렇게 노드에 직접 트랜잭션을 보내게 된다면 네트워크 지연 및 합의 과정 등에서 지연이 발생해 실시간성이 제한된다는 문제점이 있다. 특히 Kana와 같은 defi 거래소에서는 큰 문제가 될 수 있다.

그래도 손매매보단 낫기 때문에, 해당 라이브러리를 활용해서 전략을 구상했다.

![api](../../../assets/images/quant/kana/kanalabs.png)

카나 거래소의 문제점은

1. 카나 거래소는 자체 차트를 제공하지 않고, OKX나 Binance 등 다른 거래소의 차트를 가져와 사용하고 있다. 이로 인해 오더북과 차트 간의 불일치가 발생하여, 거래자들이 실시간 시장 상황을 정확하게 파악하기 어렵다.

2. 유동성이 매우매우 부족하다. 따라서 오더북 간격 간의 스프레드도 크고, 지정가 주문이 체결되기 위해서는 시간이 오래 걸린다. 



이를 반영해 사용한 전략은 간단히 2가지였다.

1. 시장가 활용 전략

거래량을 많이 내야 할 때는 손실량이 좀 높더라도 시장가로 긁어서 최대한 빨리 거래량을 높이기 위함

2. 지정가 활용 전략

시장에 현재 유동성이 높을 때 (시장가로 긁는 사람이 많은 시간대가 있음), 이 때 지정가 주문을 넣어 놓으면 알아서 다른 사람들이 시장가로 긁어준다. 

거래량 대비 손실량이 아주 낮고 안정적이다. 



처음 2주동안은 지정가 거래만 진행하다가, 마지막 3-4일 간은 빠르게 거래량을 올리기 위해 시장가 전략을 이용하였다. 



전략 개발 다 하고, ec2 인스턴스 하나 파서 지갑별로 24시간 돌리기 시작하였다. 

![ec2](../../../assets/images/quant/kana/ec2.png)



마지막 3일 정도 남은 시점에, 슬슬 거래량을 치고 올라오는 사용자들을 볼 수 있었다. 

![network](../../../assets/images/quant/kana/network.png)

네트워크를 따 보니 오는 응답에서 지갑 주소를 찾을 수 있었다. 해당 지갑 주소들을 Aptoscan에 찍어보면서, 전략적으로 내 등수를 조절했다. 

예를 들어 현재 활발히 거래를 하고 있는 유저들은 얼마나 있는지, 어떤 전략을 사용하는지 대략적으로 유추해 거래량을 조절했다. 



투자 금액

**100만원**



손실 금액(트레이딩 비용)  

<span style="color:red;">**-50만원**</span>



수익  

<span style="color:green;">**192만원**</span>



순수익  

<span style="color:green;">**+142만원**</span>  

<span style="color:green;">**+142%**</span>



또 좋은 성과가 나온 프로젝트가 있다면..기록하겠다.



### Github Link

[github](https://github.com/Kang-heeju/kana-MarketMaking)

