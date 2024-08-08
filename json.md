```json
{
    "channel": "TJS",
    "epduResults": [
        {
            "apid":2721,
            "extraInfo":{
                "success":true
            },
            "parameterNum":2,
            "parameterResults":[
                {
                    "code":"P_TF_WSBW",
                    "enum":false,
                    "name":"识别码",
                    "origData": "0x0031",
                    "overrun": false,
                    "result": "0x0031",
                    "value":"ox0031",
                    "valueType":"OTHER"
                },
                {
                    "code":"P_TF_WSBW2",
                    "enum":false,
                    "name":"识别码2",
                    "origData": "0x0031",
                    "overrun": false,
                    "result": "0x0031",
                    "value":"ox0031",
                    "valueType":"OTHER"
                }
            ]
        },
           {
            "apid":2722,
            "extraInfo":{
                "success":true
            },
            "parameterNum":1,
            "parameterResults":[
                {
                    "code":"P_TF_WSBW3",
                    "enum":false,
                    "name":"识别码3",
                    "origData": "0x0031",
                    "overrun": false,
                    "result": "0x0031",
                    "value":"ox0031",
                    "valueType":"OTHER"
                }
            ]
        }
    ]
}
```

### 输入

参数

参数是一次性传输当前动作的所有参数吗？

怎么判断是哪个任务的哪个动作apid？

通道是否有影响



### 动作

动作时顺序的，只有完成了某个阶段的某个动作才能执行下一步





### 输出

判断依据，全部达成条件则 状态为1，有一个状态是0则状态为0

状态全为1时则  阶段状态为1

