OKEX官方有提供完整的SDK，方便经验不是很丰富的开发者调用
项目地址是：https://github.com/okex/V3-Open-API-SDK

这里以获取交割合约BTCUSD数据为例

1、合约ID会随着交易而变化，首先要解决合约ID问题
```
import requests
import re
 
symbol = 'BTC-USD-'  # 用于获取合约ID，末尾一定要加上-，否则会匹配到USDT
cycle = 'quarter'  # this_week,next_week,quarter
 
 
# 获取BTC-USD季度交割合约ID
def get_okex_instrument_id(symbol, cycle):
    url = 'https://www.okex.com/api/futures/v3/instruments' #国内机器修改为okex.me
    res = requests.get(url=url)
    instrument_ids = re.findall('({".*?)},', res.text, re.S)
    for instrument_id in instrument_ids:
        if symbol in instrument_id and cycle in instrument_id:
            instrument_id = instrument_id.split('"')[3]
            return instrument_id
```
2、使用SDK获取K线
```
import okex.futures_api as future
import pandas as pd
 
 
# 获取okex的k线数据
def get_okex_candle_data(fetureAPI, instrument_id, k_time):
    results = fetureAPI.get_kline(instrument_id, k_time)
    df = pd.DataFrame(results, dtype=float)
    df.rename(columns={0: 'MTS', 1: 'open', 2: 'high', 3: 'low', 4: 'close', 5: 'volume'}, inplace=True)
    df['MTS'] = pd.to_datetime(list(df['MTS'])).tz_convert('Asia/Shanghai').strftime("%Y-%m-%d %H:%M:%S")
    df['candle_begin_time'] = pd.to_datetime(df['MTS'], format='%Y-%m-%d %H:%M:%S')
    df = df.sort_values(by="candle_begin_time")
    df = df.reset_index()
    df = df[['candle_begin_time', 'open', 'high', 'low', 'close', 'volume']]
 
    return df
 
 
if __name__ == '__main__':
    apiKey = '************'
    secret = '************'
    password = '************'
    symbol = 'BTC-USD-'  # 用于获取合约ID，末尾一定要加上-，否则会匹配到USDT
    cycle = 'quarter'  # this_week,next_week,quarter
    instrument_id = get_okex_instrument_id(symbol, cycle)
    k_time = 60 # 单位为秒，60/180/300/900/1800/3600---
    fetureAPI = future.FutureAPI(apiKey, secret, password, True)
    df = get_okex_candle_data(fetureAPI, instrument_id, k_time)
    print(df)
```

3、注意

通常情况下获取到的最后一根K线是不完整的，入库时候要注意处理，否则会导致数据不准确，比如用df.drop掉最后一条数据。

4、其它说明

SDK中默认OKEX域名是www.okex.com，在consts.py中可以修改，国内机器修改成www.okex.me

OKEX提供的数据量是有限制的，比如交割合约5分钟数据只提供最近的300条，可以改一改代码放云上定时跑用来收集数据，我从18年开始跑的，目前数据量已经满足回测。

SDK详细使用可以参考项目中的示例，配合官方API文档更容易使用，地址是 https://www.okex.me/docs/zh/

 

5、再来一波

各个交易所最多只能提供最近2000条数据，粒度越大提供的条数越少。

博主这里长期采集OKEX、火币、币安的分钟级别数据，网站上面有数据说明 http://quantdata.top  需要的联系我微信wh909077093，备注（k线），比市面上合适多的多哦。
