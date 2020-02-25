# 安装轻节点
### 获取安装程序
下载地址：
`https://github.com/barkisnet/barkisnet-binary/raw/master/barkisnet-mainnet/binary/v2.2.1/barkiscli`

### 启动
`nohup ./barkiscli rest-server --chain-id barkisnet-dev --trust-node --node 18.176.60.252:26657 --laddr tcp://0.0.0.0:1317 > lcd.log 2>&1 &`

> 1. `--chain-id barkisnet-testnet` 指定接入的网络chain-id
> 2. `--laddr tcp://0.0.0.0:1317` 参数指定绑定当前轻节点服务器的公共IP，`0.0.0.0`代表可以被其它机器访问，假如想要只能本机程序来访问，需要改成`127.0.0.1`。
> 3. `--trust-node --node 18.176.60.252:26657` 是接入测试网络的入口IP
> 4. 生产环境参数：`nohup ./barkiscli rest-server --chain-id barkisnet --trust-node --node 18.176.236.231:26657 --laddr tcp://0.0.0.0:1317  > lcd.log 2>&1 &`
> 
**注意事项**

1. 提供的安装程序是在ubuntu 18.04上编译通过的，所以服务器操作系统务必是ubuntu 18.04；
2. 由于轻节点上存有私钥，所以轻节点不能被外网访问；
3. 对于准备废弃轻节点服务器，务必执行删除私钥的操作；
4. 账号的私钥默认存在$HOME/.barkiscli目录下，一旦确定不适用轻节点了，请删除这个目录，删除前，注意备份好私钥/助记词；

# 接口介绍
## 1. 创建新账户
 **接口名称：** `/keys`
 
 **执行示例：**
`curl -X POST "http://{轻节点IP}:1317/keys" -H "accept: application/json" -H "Content-Type: application/json" -d "{ \"name\": \"my-wallet-demo123\", \"password\": \"barkisnet\"}"`
>name账户名称，password是密码

 **返回内容：**
```
{
  "name": "my-wallet-demo123",
  "type": "local",
  "address": "barkis1guv9p7xpmsug9avlcvy4lqcysyuhwfv7pmq0sr",
  "pub_key": "barkispub1addwnpepqgsdzltw2z29jz60t5lhecdp9k63e3xaj5g53cp22h2a6twp35gfcne65yz",
  "mnemonic": "stand smart quality casual tunnel husband enable doctor almost foster wink someone"
}
```
>address字段是新账户的地址，mnemonic是新账户的助记词

## 2. 使用助记词导入老账户
**接口名称：** /keys/{name}/recover

**执行示例：**
`curl -X POST "http://{轻节点IP}:1317/keys/my-demo-wallet/recover" -H  "accept: application/json" -H  "Content-Type: application/json" -d "{  \"password\": \"barkisnet\",  \"mnemonic\": \"elephant point wonder float face cool blame laundry fault moon twist muffin\”}"`
>password是密码，mnemonic是你自己的助记词

**返回内容：**
```
{
  "name": "my-demo-wallet",
  "type": "local",
  "address": "barkis1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860",
  "pub_key": "barkispub1addwnpepqdj3umzkzq3fl4rfm7vfhrzkylhq6evvp7z6avel9lxzp4xa2e8d5zt8m69"
}
```
>name账户名称
>address是账户地址

## 3. 获取账号信息接口
**接口名称：** /auth/accounts/{address}

**执行示例：**
`curl -X GET "http://{轻节点IP}:1317/auth/accounts/barkis1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860" -H  "accept: application/json"`

**返回内容：**
```
{
  "height": "329547",
  "result": {
    "type": "cosmos-sdk/Account",
    "value": {
      "address": "barkis1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860",
      "coins": [
        {
          "denom": "ubarkis",
          "amount": “1000000000" 
        }
      ],
      "public_key": null,
      "account_number": "55",
      "sequence": "0"
    }
  }
}
```
>denom是固定字符串“ubarkis”；

>amount是当前账户的余额，需要除以```10^6```，才是展示给用户看到的BKS数量；

>account_number 与 sequence 参数是留给转账接口使用的，所以在每次转账之前，都需要调用这个接口获取这两个参数，然后再调用转账接口。

## 4. 转账接口
**接口名称：** /bank/accounts/{address}/transfers

**执行示例：**
`curl -X POST "http://{轻节点IP}:1317/bank/accounts/barkis1csl7cm802trp2lqdky3ngpl9xka8av7y28s9ug/transfers" -H  "accept: application/json" -H  "Content-Type: application/json" -d "{  \"base_req\": {    \"from\": \"barkis1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860\",    \"password\": \"barkisnet\",    \"memo\": \"first transfer to a account\",    \"chain_id\": \"barkisnet-dev\",    \"account_number\": \"55\",    \"sequence\": \"0\",    \"gas\": \"100000\",    \"gas_adjustment\": \"1.2\",    \"fees\": [      {        \"denom\": \"ubarkis\",        \"amount\": \"1000\"      }    ],    \"broadcast_mode\": \"sync\",    \"simulate\": false  },  \"amount\": [    {      \"denom\": \"ubarkis\",      \"amount\": \"22000000\"    }  ]}"`
>address：收款方地址；

>account_number：账户编号，一旦创建了账户，这个值是不变的；

>sequence：每次转账都需要从/auth/accounts/{address}接口获取最新的值；

>from：发送方地址；

>broadcast_mode：转账模式，可选的参数sync（阻塞），async（不阻塞）；

>amount：转账数量，bks公链的所有数值，通过接口上链的时候，需要在1个BKS单位的基础上乘以 ```10^6```, 从接口中拿到的数据也要除以 ```10^6``` 后展示给用户，比如你想转20个BKS，则amount的值如下
```math
20 * 10^{6} = 20,000,000
```


**返回内容：**
```
{
  "result": {
    "raw_log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\"}]",
    "txhash": "9CE5BE6A700A6199811204B473188BE94089963D5CBEB39E3493F92B87E3CCA0",
    "height": "0",
    "logs": [
      {
        "success": true,
        "log": "",
        "msg_index": 0
      }
    ]
  },
  "height" : "0"
}
```
> success字段表明交易成功提交到网络，不能通过这个字段来判断交易是否成功；

> txhash是交易的hash值，需要通过/txs{hash}接口来查询交易的详细信息；

## 5. 根据转账返回的hash值，查询单条交易
**接口名称：** `/txs/{hash}`

**执行示例：**
`curl -X GET "http://{轻节点IP}:1317/txs/9CE5BE6A700A6199811204B473188BE94089963D5CBEB39E3493F92B87E3CCA0" -H  "accept: application/json"`

**返回内容：**
```
{
  "height": "329655",
  "txhash": "9CE5BE6A700A6199811204B473188BE94089963D5CBEB39E3493F92B87E3CCA0",
  "raw_log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\"}]",
  "logs": [
    {
      "msg_index": 0,
      "success": true,
      "log": ""
    }
  ],
  "gas_wanted": "100000",
  "gas_used": "39320",
  "events": [
    {
      "type": "message",
      "attributes": [
        {
          "key": "action",
          "value": "send"
        },
        {
          "key": "sender",
          "value": "barkis1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860"
        },
        {
          "key": "module",
          "value": "bank"
        }
      ]
    },
    {
      "type": "transfer",
      "attributes": [
        {
          "key": "recipient",
          "value": "barkis1csl7cm802trp2lqdky3ngpl9xka8av7y28s9ug"
        },
        {
          "key": "amount",
          "value": "22000000ubarkis"
        }
      ]
    }
  ],
  "tx": {
    "type": "cosmos-sdk/StdTx",
    "value": {
      "msg": [
        {
          "type": "cosmos-sdk/MsgSend",
          "value": {
            "from_address": "barkis1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860",
            "to_address": "barkis1csl7cm802trp2lqdky3ngpl9xka8av7y28s9ug",
            "amount": [
              {
                "denom": "ubarkis",
                "amount": "22000000"
              }
            ]
          }
        }
      ],
      "fee": {
        "amount": [
          {
            "denom": "ubarkis",
            "amount": "1000"
          }
        ],
        "gas": "100000"
      },
      "signatures": [
        {
          "pub_key": {
            "type": "tendermint/PubKeySecp256k1",
            "value": "A2UebFYQIp/Uad+Ym4xWJ+4NZYwPha6zPy/MINTdVk7a"
          },
          "signature": "FntwNT0aVDp+fqGXThmjplWdki5s5JyNhcdIsnnfJ5IQaoxrF398wWsKOwAWcFeGzGxtbiQdhGPXBc6wPhRcXg=="
        }
      ],
      "memo": "first transfer to a account"
    }
  },
  "timestamp": "2019-11-26T09:00:25Z"
}
```
>success：转账是否成功；

>from_address：发送方地址；

>to_address：接收方地址；

>tx->value->msg->value-amount->amount：交易的金额，需要除以```10^6```后展示给客户；

>memo：交易备注，通过这个来区分用户；

>timestamp：交易发生的时间，注意这是0时区的时间，可以根据客户端的业务需求格式化为东8区；
>

## 6. 根据发送方地址，查询交易：
**接口名称：** /txs

**执行示例：**
`curl -X GET "http://{轻节点IP}:1317/txs?message.action=send&message.sender=barkis1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860&page=1&limit=10" -H  "accept: application/json"`

**返回内容：**
```
{
  "total_count": "1",
  "count": "1",
  "page_number": "1",
  "page_total": "1",
  "limit": "10",
  "txs": [
    {
      "height": "329655",
      "txhash": "9CE5BE6A700A6199811204B473188BE94089963D5CBEB39E3493F92B87E3CCA0",
      "raw_log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\"}]",
      "logs": [
        {
          "msg_index": 0,
          "success": true,
          "log": ""
        }
      ],
      "gas_wanted": "100000",
      "gas_used": "39320",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "action",
              "value": "send"
            },
            {
              "key": "sender",
              "value": "barkis1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860"
            },
            {
              "key": "module",
              "value": "bank"
            }
          ]
        },
        {
          "type": "transfer",
          "attributes": [
            {
              "key": "recipient",
              "value": "barkis1csl7cm802trp2lqdky3ngpl9xka8av7y28s9ug"
            },
            {
              "key": "amount",
              "value": "22000000ubarkis"
            }
          ]
        }
      ],
      "tx": {
        "type": "cosmos-sdk/StdTx",
        "value": {
          "msg": [
            {
              "type": "cosmos-sdk/MsgSend",
              "value": {
                "from_address": "barkis1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860",
                "to_address": "barkis1csl7cm802trp2lqdky3ngpl9xka8av7y28s9ug",
                "amount": [
                  {
                    "denom": "ubarkis",
                    "amount": "22000000"
                  }
                ]
              }
            }
          ],
          "fee": {
            "amount": [
              {
                "denom": "ubarkis",
                "amount": "1000"
              }
            ],
            "gas": "100000"
          },
          "signatures": [
            {
              "pub_key": {
                "type": "tendermint/PubKeySecp256k1",
                "value": "A2UebFYQIp/Uad+Ym4xWJ+4NZYwPha6zPy/MINTdVk7a"
              },
              "signature": "FntwNT0aVDp+fqGXThmjplWdki5s5JyNhcdIsnnfJ5IQaoxrF398wWsKOwAWcFeGzGxtbiQdhGPXBc6wPhRcXg=="
            }
          ],
          "memo": "first transfer to a account"
        }
      },
      "timestamp": "2019-11-26T09:00:25Z"
    }
  ]
}
```
>需要关注的字段与`/txs/{hash}`接口相同

## 7. 根据接收方地址，查询交易：
**接口名称：** `/txs`

**执行示例：**
`curl -X GET "http://{轻节点IP}:1317/txs?message.action=send&transfer.recipient=barkis1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860&page=1&limit=10" -H  "accept: application/json"`

**输出内容：**
```
{
  "page_total": "1",
  "page_number": "1",
  "txs": [
    {
      "height": "328473",
      "txhash": "56CEBE1522F737F0A5F2AF1EF322A282C2ACA231C70F466A33FE31356B812C2B",
      "logs": [
        {
          "success": true,
          "log": "",
          "msg_index": 0
        }
      ],
      "gas_used": "42633",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "action",
              "value": "send"
            },
            {
              "key": "sender",
              "value": "barkis15cqg8pmmz7rdm3wkea3z4x475f3ngp0lq2n405"
            },
            {
              "key": "module",
              "value": "bank"
            }
          ]
        },
        {
          "type": "transfer",
          "attributes": [
            {
              "key": "recipient",
              "value": "barkis1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860"
            },
            {
              "key": "amount",
              "value": "1000000000ubarkis"
            }
          ]
        }
      ],
      "gas_wanted": "200000",
      "tx": {
        "type": "cosmos-sdk\/StdTx",
        "value": {
          "signatures": [
            {
              "pub_key": {
                "type": "tendermint\/PubKeySecp256k1",
                "value": "A0uAEtIIQHLNw0Hqed9fRc8lhGo7oBnTu1WJKzSBp2rg"
              },
              "signature": "VR4r9nK2i9pa\/\/zMhuEcWJejBZZz0BjOnEeZzUhxfAMHrPpgi37fftbNaydGmfANDP3QaITDr9NHvkClgd4zEA=="
            }
          ],
          "memo": "Sent token from light node",
          "msg": [
            {
              "type": "cosmos-sdk\/MsgSend",
              "value": {
                "from_address": "barkis15cqg8pmmz7rdm3wkea3z4x475f3ngp0lq2n405",
                "amount": [
                  {
                    "denom": "ubarkis",
                    "amount": "1000000000"
                  }
                ],
                "to_address": "barkis1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860"
              }
            }
          ],
          "fee": {
            "amount": [
              {
                "denom": "ubarkis",
                "amount": "1000000"
              }
            ],
            "gas": "200000"
          }
        }
      },
      "timestamp": "2019-11-26T07:20:30Z",
      "raw_log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\"}]"
    }
  ],
  "total_count": "1",
  "count": "1",
  "limit": "10"
}
```
>需要关注的字段与`/txs/{hash}`接口相同
>
# 三. 如何集成
>主要说明如何集成充值与提币，barkis链建议通过tx的memo字段来区分不同的用户，所以需要在系统层面为每个客户设置一个唯一的ID标识，一般用数字来表示

## 1.充值
在充值页面需要给用户展示官方钱包的充值地址与需要用户填入的memo内容。系统需要查询自己地址的转入交易，然后通过查看tx的memo字段来确定是哪一位用户执行了充币。
&nbsp; &nbsp; 
## 2.提币
在提币页面需要让用户输入提币地址与memo（假如客户从当前交易所提到别的交易所，则需要填写memo字段；若是提到自己的去中心钱包，则不需要填写memo字段）。

#### 提币需要用到转账接口，转账的基本调用逻辑：
> a. 先调用获取账号信息的接口，提取account_number与sequence参数；

> b. 把上面两个参数放到转账接口中，编排好接口的body参数的json结构，然后执行；
