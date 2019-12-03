---
layout: post
title: 업비트 해킹 트랜젝션에 대한 고찰
tags: [cryptocurrency, security, blockchain]
---

업비트가 털렸다. 뭐 그건 중요한 건 아니고, 아니 뭐 ISMS 인증 받고 철통 보안이라던데가 털렸고, 그것도 핫 월렛 하나가 통째로 날라갔으니 얼척이 없다고 하는게 더 낫겠지만, 어쨌든 정말로 흥분되는 일이긴 하다. 남의 초상집 앞에서 흥분된다고 하기에는 좀 그렇지만, 이  해킹에 있어서 재미있는 흔적들이 너무나도 많이 남았기 때문에 이 이야기를 하다보면 흥분이 될 수 밖에 없기 때문일 것이다.

342,000 이더라는 사상 초유의 이더리움 해킹 사태는 둘째치고, 업비트 해킹 트랜젝션을 분석하면 재미있는 사실들을 많이 알 수 있다. 이는 내부자의 치밀한 소행이거나, 외부에서 각을 재고 있었다가 공격을 들어온 케이스라고 생각이 되는 부분인데, 이를 하나씩 풀어가면서 이야기를 해보도록 하자.

다음은 문제의 그 이더리움 트랜젝션이다. ([이더스캔](https://etherscan.io/tx/0xca4e0aa223e3190ab477efb25617eff3a42af7bdb29cdb7dc9e7935ea88626b4))

```json
$ curl -H "Content-Type: application/json" --data "{\"jsonrpc\":\"2.0\",\"method\":\"eth_getTransactionByHash\",\"params\":[\"0xca4e0aa223e3190ab477efb25617eff3a42af7bdb29cdb7dc9e7935ea88626b4\"],\"id\":67}" http://localhost:8645 | jq

{
  "jsonrpc": "2.0",
  "result": {
    "blockHash": "0xd7b9bc7995cefc295fd62fd09ebccf978ed935d0e7ebe5fdf70e1e2ce793bd9f",
    "blockNumber": "0x8972f7",
    "chainId": "0x1",
    "condition": null,
    "creates": null,
    "from": "0x5e032243d507c743b061ef021e2ec7fcc6d3ab89",
    "gas": "0x30d40",
    "gasPrice": "0xe8d4a51000",
    "hash": "0xca4e0aa223e3190ab477efb25617eff3a42af7bdb29cdb7dc9e7935ea88626b4",
    "input": "0x00",
    "nonce": "0x8f7b4",
    "publicKey": "0x58ed29a99e06f14a5b7dc33a3d670f3274dfa7ebe891c20e3b8df2aa5d1de12676a3a8ca747d703db52126197a743535640450bb1b89c68a3848129fe5751ac3",
    "r": "0x551f02b088b2f8d99522c5411ff44926ca73f0bac7413db6a81aebfc32844bce",
    "raw": "0xf8728308f7b485e8d4a5100083030d4094a09871aeadf4994ca12f5c0b6056bbd1d343c0298a486bdb6e265769c000000025a0551f02b088b2f8d99522c5411ff44926ca73f0bac7413db6a81aebfc32844bcea01aaad47971f4a96c40645072604f7268500e1ed046dac9742482d904471ad176",
    "s": "0x1aaad47971f4a96c40645072604f7268500e1ed046dac9742482d904471ad176",
    "standardV": "0x0",
    "to": "0xa09871aeadf4994ca12f5c0b6056bbd1d343c029",
    "transactionIndex": "0x0",
    "v": "0x25",
    "value": "0x486bdb6e265769c00000"
  },
  "id": 67
}
```



### 1. tx nonce가 상당히 의미 심장하다. (587700 번)

실제로 내부자가 핫 월렛에 접속을 하여 바로 트랜젝션을 땡겼다는 가정을 한다면, 0이 2개씩이나 포함되는 딱 떨어지는 이쁜 숫자가 나오기는 힘들다. nonce는 외부 출금이 한 번씩 일어날 때마다 증가하는 숫자인데, 공격자가 587700번째(정확히는 587701번째, 프로그래머는 0부터 센다)에 의도적으로 인출을 할 생각을 할 것인가? 미묘하다.

### 2. Gas Limit를 임의적으로 주었다.

일반적으로 Account간 이더리움을 전송하는 트랜젝션에 소요되는 Gas비는 21,000 정도이다. 이에 따라서 업비트는 45,000 정도를 Gas Limit을 걸어놓고 쓰고 있는데, 공격자는 200,000으로 Gas Limit을 잡았다. 일반적으로 이더리움 핫 월렛에서 직접 조작을 하거나, 트랜젝션을 생성하는 전용 서버가 털렸을 경우에는 200,000이라는 값이 나오기는 힘들 것이다. 실제로 이 때문에, 내부적으로 툴이나 스크립트를 돌려서 공격을 했다는 심증을 갖게 해준다.

### 3. Data 필드에 실수인지 고의인지 데이터를 넣었다

0x00 라는 데이터를 넣었는데, 일반적으로 Data는 트랜젝션을 통해서 스마트컨트랙트의 함수 콜을 위해서 사용하는 란으로, 일반적인 계정간 이동에서는 사용되지 않는 란이다. 보통, 일반적인 트랜젝션에 Data에 값을 넣는다면 [ASCII로 데이터를 넣어서 블록체인 상에 텍스트를 기록하는데 사용](http://www.hashedpost.com/2018/04/hashed-report-4-23.html)을 하는데, 이것도 아닌 0x00라는 미묘한 값이 들어갔다.

트랜젝션을 생성하면서, 공격자가 실수로 0x00라는 값을 넣었을 가능성이 있다. 초보적인 실수로 생각을 하고 있는데, 4/5번과 같이 보면 의도적일 가능성도 충분히 존재한다.

### 4. Gas Price가 상당히 높다.

일반적으로 이더리움에서는 10 Gwei 정도의 Gas Price를 넣어도 1분 내에 트랜젝션이 처리된다. (5~7 Gwei 정도여도 무방하다. [Ethergas station](https://ethgasstation.info/)을 참고하자) 그런데 Gas Price를 무려 1,000 Gwei, 통상 업비트의 60 Gwei의 15배 이상을 넣었다. (Gas Price는 우편/당일특송/퀵 이런 느낌으로 생각하면 편하다. 가격이 높을수록 빨리 전송이 일어난다. 당연히 블록 생성 속도보다는 빠를 수 없다.)

단순하게 생각한다면, 쓸데없이 값을 높게 책정을 했다는 것이고, 좀 생각을 하자면, 그만큼 급했다는 것을 알 수 있다. 아니면, 다른 의도를 갖고 있거나.

### 5. 왜 342,000 이더인가?

실제로 4,200 이더 정도를 더 가져갈 수 있었음에도 불구하고 공격자는 한 번에 342,000 이더만 빼갔다. 뭐 이미 90% 이상의 금액을 빼갔으니 별 문제는 없다고 생각을 하겠지만, 그 당시에 핫월렛에 직접 접근이 가능했다면 바로 전액을 자기가 원하는 주소로 전송을 했을텐데 그렇지 않았다는 점이다. 4,200이더면, 8억원 정도 되는 금액인데 알뜰살뜰 하게(?) 그 금액까지 빼 내가지 않았다는 점이 수상하다.

### 예상 되는 시나리오

1/2/3/4/5를 같이 본다면, 실제로 뭔가 미묘하게 이상하다. 통상적인 절차라면 해커는 핫 월렛 노드까지 접근을 한 뒤 바로 최대 금액을 자기 자신의 지갑으로 땡겨서 넣는 것이 상당히 정석적인 방법이었으리라고 본다. 하지만, 그렇지 않았다는 점은 핫 월렛에 직접적으로 접근해서 공격한 것이 아니라, 뭔가 시스템을 타고 타고 들어가서 PK를 빼냈거나, APT로 노드에 접근을 한 뒤 노드와 통신을 JSON-RPC로 트랜젝션을 삽입 했던게 아닌가 싶다. 그 와중에 쉘 스크립트나 공격용 코드를 사용했으리라 생각을 하는데, 그 이유는 1. nonce 값이 작위적이고 (코드로 짰고) 2. Gas Limit의 경우 만일을 대비해서 높게 책정했고  3. Data 필드에 실수로 0x00을 넣었고 4. Gas Price도 1,000 Gwei를 하드코딩했을 가능성이 크다. (Gas Price는 변동이 있을 수 있으니 그냥 1000을 박아넣었던 것) 최종적으로, 일주일이나 한 달 동안 잔고 평균을 내서 이 정도면 괜찮으리라 생각한 342,000 이더만 빼갔을 것이라는 것이다.

바로 핫 월렛에서 붙어서 공격을 한게 아니라, 직접 접근이 불가능한 상태에서 추측만으로 공격용 툴이 타고 타고 가능성이 제일 높다고 본다.

### 공격자의 운빨? 아님 의도?

거기서 운이 좋은 건지, 나쁜건지는 모르겠지만, 공격자는 트랜젝션이 덮어씌워질 수 있다는 것을 충분히 알고 있었던 것으로 보인다. data 값의 0x00은 무의미한 값이고, Gas Price의 1000 Gwei는 너무 높은 값이지만, 트랜젝션을 덮어 씌울 수 있다는 점을 안다면 이것은 상당히 노리고 뭔가를 했다는 점일 것이다.

실제로 [트랜젝션을 취소 할 수는 없지만 덮어 씌우는 것은 가능하다는 것](https://ethereum.stackexchange.com/questions/31298/is-it-possible-to-cancel-a-transaction) 은 꽤 오래전부터 알려져있는 사실이다. Metamask에서도 시험적으로 도입을 하였는데, 원리는 간단하다.

이더리움 트랜젝션의 nonce 값은 전송을 할 때마다 하나씩 늘어난다. 그러니 1번째 전송, 2번째 전송, 3번째 전송... 이런식으로 값이 늘어나는데, 이 경우 이더리움 트랜젝션 내에 nonce 필드 값이 변하게 되는 것이다. 이 때, nonce 2의 값을 갖는 트랜젝션을 새로 생성해 전송 시키면 트랜젝션을 페일이 난다. 그렇다면, nonce 4의 값을 갖는 다른 종류의 트랜젝션을 2개 생성해서 노드에 쏘면 어떻게 될까?

일반적으로, 이것을 이중 지불 공격이라고 한다. 다행히도 블록체인에서는 이것을 해결을 다음과 같이 한다. 두 사람에게 각각 이더를 보내는 다른 트랜젝션을 만들게 되면, 노드들은 그 트랜젝션 둘 중 하나를 선택하게 된다. 일반적으로, 채굴자 노드들은 (가성비가 높은) 수수료(Fee)가 높은 것을 택하는 습관이 있다. Gas Price * Gas used 값이 높은 것을 택하는 건데, 간단히 설명하면 500원짜리 우표보다 3500원짜리 등기가 더 선택된다는 것이다. 즉 상반되는 두 트랜젝션이 생성되더라도, 둘 중 하나를 택하는 것을 어느정도 예측 가능하고, 이를 통해서 블록체인이 운영된다는 것이다. 그리고 이를 통해서, Gas Price가 더 높은 트랜젝션을 같은 nonce로 발행을 하면 이를 덮어 씌워서 이전 트랜젝션을 취소 시킬 수 있는 것이다.

그렇기에 1000 Gwei짜리 트랜젝션을 만들어낸 것은, 트랜젝션이 덮어 씌워짐을 막기 위함이라는 것을 유추할 수 있다. 빠르게 전송을 시킬려면 100 Gwei 만 줬어도 별 문제가 없었을 텐데, 이렇게 1000 Gwei를 준 것은 의도적이었다는 것이다.

또한 Data 필드에 0x00도 동일하게 해석할 수 있다. 트랜젝션의 사이즈를 늘려서, 트랜젝션이 덮어 씌우는 트랜젝션 대신에 선택 되도록 하는 꼼수였을 가능성이 크다. 0x00을 통해서 21,00__4__ 의 가스량이 소모가 되었고, +4 덕분에 21,000 짜리 트랜젝션보다 더 선택이 될 가능성이 높은 것이다. (뭐 엄밀히 말하자면, 이것은 옳은 설명은 아니다. 택배처럼 무게랑 부피랑 가격 3가지 요소가 영향을 미치듯이, 트랜젝션도 이 외의 영향을 주는 요소가 많다.)

즉, 이 트랙젝션 하나하나에는 모두 공격자가 빠르고 안전하고 실수 없이 외부로 적절한 금액의 이더를 반출하려는 노력이 담겨 있다는 것이다.

### 그래서?

이렇다면, 서버 어딘가에 이 특이한 모양의 트랜젝션을 생성한 흔적이 있을 것이다. 아마도. 특히 JSON-RPC 콜에 대한 로깅을 하고 있다면, 이 로그가 핵심일 것이다.

### 그렇다면 이걸 나중에 막을 방법은?

사실 이상 트랜젝션 검출을 하면 된다. 일단 PK에 대응하는 어드레스의 트랜젝션들을 계속 감시하면서 이상 트랜젝션이 발생하면, 앞에서 말한 것처럼 가스비를 더 높게 주는 같은 nonce 값을 갖는 트랜젝션을 생성해서 트랜젝션을 덮어 씌우고, 잠시 입출금을 동결하거나 서버 점검을 돌리면 될 것이기 때문이다.

즉, 트랜젝션의 Gas Price, 금액, Data 필드만 제대로 감시를 했어도 바로 대응이 가능했으리라고 보고 있다.

### 결론은?

보면 실수처럼 보이지만, 의도는 분명한 것들이 있다. 초보자라는 느낌이 들지만, 그렇다고 해킹의 초보는 아니고, 아마 블록체인을 그렇게까지는 잘 알고 있는 타입까지는 아닌 것 같다. 사실, 나 같았으면 Gas Price를 100 Gwei를 넣고 Data 필드는 비워놨을 것이다. (그래도 별 문제는 없다.) 아님 한 번에 10~20 이더를 출금하는 대량의 (팬딩) 트랜젝션을 발생시키거나 하는 방식으로 침임 탑지가 되나 막기가 까다로운 방식으로 공격을 감행했을 것이다. 하지만, 요번 공격은 흔적이 덜 남는 쪽으로 공격을 했어야했는데, 이미 충분한 시그니처가 남아버렸고, 이미 어드레스에 대한 추적도 이루어지니, DEX를 통하거나 토큰으로 스왑하지 않는 이상 현금화는 어려우리라 보고 있다.

(아님, Ethereum 2.0 Deposit Contract에 이체하던지?)