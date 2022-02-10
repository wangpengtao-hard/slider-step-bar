# slide-step-bar
## 运行项目
npm install  
npm start

## 配置项
currentDate             // 当前日期展示（非必填 13位时间戳） 默认 当前日期  
compareData             // 用户传入的数据，原样抛出，不作任何处理  
roomList                // 和已选中的数据（非必填 array)  
                        [  
                            {  
                                conference_room_name: '房间 1',  
                                reserveInfoList: [  
                                    {  
                                        startTime: '1642226400000', // 14:00  
                                        // endTime: '1642228200000', // 14:30  
                                        // endTime: '1642230000000', // 15:00  
                                        endTime: "1642235400000", // 16:30  
                                        reserveLabel: '赵晓阳'  
                                    }  
                                ]  
                            },  
                            {  
                                conference_room_name: '房间 2',  
                                reserveInfoList: [  
                                    {  
                                        startTime: '1642226400000',  
                                        endTime: '1642228200000',  
                                        reserveLabel: '123'  
                                    }  
                                ]  
                            }  
                        ]  
dataSource              // 数据源 (非必填 array) 默认8-23  [8, 8.5, 9, .... 23]  

## 事件
@confirm 点击确定

##方法
reset() // 重置


## demo

