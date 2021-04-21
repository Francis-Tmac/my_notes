### Prometheus
- 渠道维度超时统计/天
``` 
metric-1:
sum(increase(xiaoying_afp_fund_hub_fund_interaction_time_count{interaction_name=~"$interaction_name",channel_code=~"$channel_code",is_timeout = 'YES'}[24h])) by (channel_code)
metric-2:
sum(increase(xiaoying_afp_fund_hub_fund_interaction_time_count{interaction_name=~"$interaction_name",channel_code=~"$channel_code",is_timeout = 'YES'}[24h]))
```


- 资方维度请求量/天

``` 
metric-1:
sum(increase(xiaoying_afp_fund_hub_fund_interaction_time_count{interaction_name=~"$interaction_name",channel_code=~"$channel_code"} [24h]))
metric-2:
sum(increase(xiaoying_afp_fund_hub_fund_interaction_time_count{interaction_name=~"$interaction_name",channel_code=~"$channel_code"} [24h]))   by (channel_code)

```



