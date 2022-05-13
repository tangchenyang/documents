## 1 创建 stream / topic
```shell
aws kinesis create-stream --stream-name test_chenyang
```
### 查看 stream / topic 状态
```shell
aws kinesis describe-stream-summary --stream-name test_chenyang
```

## 2 列出所有 stream / topic
```shell
aws kinesis list-streams
```

## 3 Put 数据 
```shell
aws kinesis put-record --stream-name test_chenyang --partition-key 101 --data '{"name": "chenyang"}'
```
将返回数据所在的shard ID
```json
{
    "ShardId": "shardId-000000000000",
    "SequenceNumber": "49629432957388521732637430547576992512317925690177486850"
}
```

## 4 Get 数据 
### 先为 stream / topic 获取对应分片的迭代器
```shell
aws kinesis get-shard-iterator --shard-id shardId-000000000000 --shard-iterator-type TRIM_HORIZON --stream-name test_chenyang
```
将返回对应的 ShardIterator
```json
{
    "ShardIterator": "AAAAAAAAAAHJoy3nzOIh/eWwegBwMnLaTCC8K1nQ5qNFm3ciAHD3F7qq9/JppjNDLENPsBt/X9XidteXFngEFBBkIGCgPQr2pIENeusrZH2StbwxGEFGAk+tDn2adGJtIIT79+2RkUSqBd0BQSe42nIIrihbcsi86KL5UrS7lQNDfspom6GXUScsmsRnNmHyqxQhfT1egZccWZl9xpzmLawDzPxu1hRAHVWiWkufsbQNeDMqBH/VQQ=="
}
```
### 使用 ShardIterator 获取数据
```shell
aws kinesis get-records --shard-iterator AAAAAAAAAAFywBsnySw1y3hgJRNWBJdJIM5bzeGFyhFmeLLd3VHWCEeeZBJF7mLOX/KR90bypxzhTDsqhvjK+YI9CnzS3xvayBOyWk/95YZd7SnsNUMrzN2GAT6pEr1+Sp6fgbczw+BX7aPhK6cZa79i2ICBWP5AE7wsD31gZTgmxxr8/riXa4P8u5/qNWVIhowDyz6bwQOsryrHOOyIkSZU1TorAGEzgqgdvPqGE799zjcTjyrw/w==
aws kinesis get-records --shard-iterator AAAAAAAAAAEnX79VsWmJV4SP1uGnaAk3Vn1vVLghefJekIob9AacWsURbTrpV9mQ8s8pAZo830IM/TCqmA/oXneFdhjJ6+5MdlTe1pgEofacyzSP1Jr+7u4wYvMVSytBgGH3S8yrP1GMOIbfVV+7AZNEw1IaylewSAmcQgTQLsSQgmV9pCnVEd0COcl6B1abVs6idoj03O7hyLRtZa6KWYBuwM9TadYsuM1ifeSbotrSR6bW67LThQ==
```
### 一步到位的脚本 
```shell
SHARD_ITERATOR=$(aws kinesis get-shard-iterator --shard-id shardId-000000000000 --shard-iterator-type TRIM_HORIZON --stream-name test_chenyang --query 'ShardIterator')

aws kinesis get-records --shard-iterator $SHARD_ITERATOR
```

## 5 删除 stream / topic
```shell
aws kinesis delete-stream --stream-name test_chenyang
```

### 查看 stream / topic 状态
```shell
aws kinesis describe-stream-summary --stream-name test_chenyang
```