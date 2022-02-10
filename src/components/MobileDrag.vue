<template>
    <div class="mobile-drag">
        <div class="mobile-drag-content">
            <div class="mobile-drag-content-item"
                v-for="(roomItem, roomInd) in localDisableData"
                :key="roomInd"
                @click="clickItem(roomItem, roomInd)"
            >
                <div class="mobile-drag-info">
                    <p class="mobile-drag-info-label">{{roomItem.conference_room_name}}</p>
                </div>
                <ul class="mobile-drag-content-slider" :ref="'mobile-drag-content-slider' + roomInd" >
                    <li v-for="(item, ind) in roomItem.dataSource" :key="item.localLabel">
                        <div
                            class="block" 
                            :class="{
                                integer: ind % 2 == 0, odd: ind % 2 == 1,
                                disabledClass: roomItem.reserveInfoList && roomItem.reserveInfoList.filter(x => x.isHistory).find(xx => {
                                    let flag = item.localLabel >= xx.step[0] && item.localLabel < xx.step[1];
                                    if (flag) {
                                        item.isDisabed = true
                                    }
                                    return flag
                                }),
                                alreadyClass: roomItem.reserveInfoList && roomItem.reserveInfoList.filter(x => !x.isHistory).find(xx => {
                                    let flag = item.localLabel >= xx.step[0] && item.localLabel < xx.step[1];
                                    if (flag) {
                                        item.isDisabed = true;
                                        item.alreadyDisalbe = true;
                                        item.alreadyInfo = xx
                                    }
                                    return flag
                                }),
                            }"
                        >
                        </div>
                        <div class="block-num">
                            <span v-show="ind % 2 == 0">{{item.localLabel}}</span>
                        </div>
                    </li>
                </ul>    
            </div>
        </div>
        <van-popup v-model="isShowSelectAndMask" position="bottom" :style="{ height: '96%', borderRadius: '8px' }" >
            <div class="content">
                <div class="header">
                    <div class="header-title">{{roomDetail.conference_room_name}}</div>
                    <div class="header-tip">请选择需预定的时间</div>
                </div>
                <!-- dataSource -->
                <div class="pop">
                    <div class="pop-list" v-for="(item, index) in roomDetail.dataSource" :key="index">
                        <div class="pop-list-label">
                            {{item.localLabel}}
                        </div>
                        <div
                            class="pop-list-item"
                            :ref="'popListItem' + index"
                            :class="{
                                integer: index % 2 == 0, odd: index % 2 == 1,
                                disabledClass: item.isDisabed,
                                alreadyClass: item.alreadyDisalbe,
                                lastItem: index == roomDetail.dataSource.length - 1,
                                allowSelectClass: item.isAllowSelect
                                
                            }"
                            @click="clickStep($event,item, index)"
                        >
                        </div>
                    </div>
                </div>
            </div>

        </van-popup>
    </div>
</template>
<script>
export default {
    name: 'mobile-drag',
    props: {
        currentDate: { // 当前日期展示（非必填） 13位时间戳 默认当前时间
            type: Number || String,
            default: new Date().getTime()
        },
        compareData: { // 用户传入的数据，原样抛出，不作任何处理
            type: Object,
            default: () => {}
        },
        roomList: { // 已选中的数据（必填）
            type: Array,
            default: () => [
                {
                    conference_room_name: '房间 1',
                    reserveInfoList: [
                        {
                            startTime: '1642240800329', // 18:00
                            // startTime: '1642226400000', // 14:00
                            // endTime: '1642228200000', // 14:30
                            // endTime: '1642230000000', // 15:00
                            // endTime: "1642235400000", // 16:30
                            endTime: "1642244400616", // 19:00
                            reserveLabel: '赵晓阳'
                        }
                    ]
                },
                {
                    conference_room_name: '房间 2',
                    reserveInfoList: [
                        {
                            // startTime: '1642240800329', // 18:00
                            startTime: '1642226400000', // 14:00
                            endTime: '1642228200000', // 14:30
                            // endTime: '1642230000000', // 15:00
                            // endTime: "1642235400000", // 16:30
                            // endTime: "1642244400616", // 19:00
                            reserveLabel: '赵晓阳赵晓阳赵晓阳赵晓阳赵晓阳赵晓阳赵晓阳赵晓阳赵晓阳赵晓阳赵晓阳赵晓阳赵晓阳赵晓阳'
                        }
                    ]
                },
                {
                    conference_room_name: '房间 3',
                    reserveInfoList: [
                        {
                            // startTime: '1642240800329', // 18:00
                            startTime: '1642226400000', // 14:00
                            endTime: '1642228200000', // 14:30
                            // endTime: '1642230000000', // 15:00
                            // endTime: "1642235400000", // 16:30
                            // endTime: "1642244400616", // 19:00
                            reserveLabel: '赵晓阳'
                        }
                    ]
                }
            ]
        },
        dataSource: { // 渲染数据源 默认 8-23 （非必填）
            type: Array,
            default: () => [
                8,
                8.5,
                9,
                9.5,
                10,
                10.5,
                11,
                11.5,
                12,
                12.5,
                13,
                13.5,
                14,
                14.5,
                15,
                15.5,
                16,
                16.5,
                17,
                17.5,
                18,
                18.5,
                19,
                19.5,
                20,
                20.5,
                21,
                21.5,
                22,
                22.5,
                23
            ]
        }
    },
    data() {
        return {
            isShowSelectAndMask: false, // 是否展示弹窗
            roomBtmTipLabel: '', // 存储下方弹窗的房间label

            dateArray: [], // [8. 9] // 当前选中的区间
            dateLabelArray: [], // [8:00, 9:30] 当前选中的区间
            
            alreadyNum: '', // 已选中的label
            alreadyLabelArray: [], // 已选中的日期存储 [11:30, 14:00]
            
            currentInd: 0, // 当前在第几个会议室

            localDisableData: [], // 当前组件存储禁用和已选中的数据
            roomDetail: {}, // 当前会议室信息
        }
    },
    methods: {
        clickStep(event, item, index) {
            if (item.isDisabed || item.alreadyDisalbe || index >= this.roomDetail.dataSource.length - 1) return
            let allowList = this.roomDetail.dataSource.filter(x => x.isAllowSelect);
            if (!allowList.length) { // 一个都没有选中
                item.isAllowSelect = true;
            } else {
                let firstSelectInd = allowList[0]['index'];
                let [lInd, rInd] = [firstSelectInd, index].sort((a, b) => a - b); // 从小到大排序
                if (allowList.length == 1 && this.allowConfirm(lInd, rInd)) { // 只选中了一个 && 允许连选
                    this.roomDetail.dataSource.forEach((x, ind) => {
                        if (ind >= lInd && ind <= rInd) {
                            x.isAllowSelect = true;
                        } else {
                            x.isAllowSelect = false;
                        }
                    })
                } else { // 已经选中了多个，（需要只选中最后点击的那个）
                    this.roomDetail.dataSource.forEach((x, ind) => {
                        x.isAllowSelect = false;
                        if (index == ind) {
                            x.isAllowSelect = true;
                        }
                    })
                }
            }
            this.getCurrentStep(); // 获取当前选中的信息
        },
        clickItem(roomItem, roomInd) {
            this.isShowSelectAndMask = true;
            this.roomDetail = roomItem;
            this.currentInd = roomInd; // 当前在第几个会议室
        },

        getCurrentStep() { // 获取当前选中的信息并发送给父级
            let startIndex = this.roomDetail.dataSource.findIndex(x => x.isAllowSelect);
            let endIndex = this.roomDetail.dataSource.findLastIndex(x => x.isAllowSelect);
            let startItem = this.roomDetail.dataSource[startIndex];
            let endItem = this.roomDetail.dataSource[endIndex + 1];

            let {year, mon, date} = this.getAssemblyDate(this.currentDate);
            let startLabel = this.assemblyTime(startItem.localLabel), endLabel = this.assemblyTime(endItem.localLabel);
            this.$emit('confirm', {
                dateParams: {
                    startComLabel: year+'-'+mon+'-'+date+' '+startLabel,
                    startTime: new Date(year+'/'+mon+'/'+date+' '+startLabel).getTime(),
                    startDate: startItem.localLabel,
                    startLabel,
                    endComLabel: year+'-'+mon+'-'+date+' '+endLabel,
                    endTime: new Date(year+'/'+mon+'/'+date+' '+endLabel).getTime(),
                    endDate: endItem.localLabel,
                    endLabel
                },
                roomInfo: this.roomList[this.currentInd],
                currentDate: this.currentDate,
                compareData: this.compareData
            })
        },
        // 设置disabled data
        setDisabledData() {
            this.localDisableData = JSON.parse(JSON.stringify(this.roomList));
            // 获取当前年月日的时间戳以及时、分
            let {ymdTime: currentDate, hour, min} = this.getAssemblyDate();

            // 获取传入的年月日的时间戳
            let {ymdTime: propDateTime} = this.getAssemblyDate(this.currentDate);

            this.localDisableData.forEach((x) => {
                x.reserveInfoList && x.reserveInfoList.forEach(xx => {
                    let {hour: startH, min: startM} = this.getAssemblyDate(xx.startTime * 1);
                    let {hour: endH, min: endM} = this.getAssemblyDate(xx.endTime * 1);

                    let startTime = startM >= 30 ? startH + '.5' : startH;

                    let endTime = '';
                    if (endM == 0) {
                        endTime = endH;
                    } else if (endM <= 30) {
                        endTime = endH + '.5';
                    } else {
                        endTime = endH * 1 + 1;
                    }
                    xx.step = [startTime, endTime];
                });
                x.dataSource = JSON.parse(JSON.stringify(this.getDataSource))
            })


            if (currentDate > propDateTime) { // 传入的是过去的时间
                // 把 8-23都禁用掉
                this.localDisableData.forEach(x => {
                    if (!x.reserveInfoList) x.reserveInfoList = [];
                    x.reserveInfoList.push({
                        step: [this.getDataSource[0]['localLabel'], this.getDataSource[this.getDataSource.length - 1]['localLabel']],  // 过去的时间,z
                        isHistory: true,
                        reserveLabel: ''
                    })
                })
            } else if (currentDate == propDateTime) { // 传入的是当前
                let stepTime = min == 0 ? hour : min <= 30 ? hour + 0.5 : hour + 1;
                this.localDisableData.forEach(x => {
                    if (!x.reserveInfoList) x.reserveInfoList = [];
                    x.reserveInfoList.push({
                        step: [8, stepTime * 1],  // 过去的时间
                        isHistory: true,
                        reserveLabel: ''
                    })
                })
            }
        },

        // 传入时间戳 获取信息(年月日时间戳，时，分)
        getAssemblyDate(t = new Date()) {
            let myDate = new Date(t);
            let year = myDate.getFullYear(),
            mon = myDate.getMonth() + 1,
            date = myDate.getDate(),
            hour = myDate.getHours(),
            min = myDate.getMinutes();

            return {
                year,
                mon,
                date,
                hour,
                min,
                ymdTime: new Date(year + '/' + mon + '/' + date).getTime(), // 年月日时间戳
            }
        },

        // 时间格式转换 传入 10  转成 10:00,  传入 10.5  转成 10:30
        assemblyTime(t) { 
            t = String(t);
            let splitT = t.split('.5');
            return splitT.length == 2 ? splitT[0] + ':30' : t + ':00';
        },

        // 判断在某个区间内是否允许连选
        allowConfirm(lInd, rInd) {
            let disableLen = this.localDisableData[this.currentInd].dataSource.slice(lInd, rInd).filter(x => x.isDisabed);
            return !disableLen.length;
        },

    },
    computed: {
        getDataSource() { // 转换数据源 把 [1, 2] 转成 [{localLabel: 1}, {localLabel: 2}]
            return this.dataSource.map((x, ind) => {
                return {
                    index: ind,
                    localLabel: x
                }
            })
        },
        fromTimeGetMonthAndDay() { // 根据时间戳获取月和日
            let {mon, date} = this.getAssemblyDate(this.currentDate);
            return mon + '月' + date + '日';
        }
    },
    mounted() {
        // 设置disabled data
        this.setDisabledData();
    },
    watch: {
        roomList: {
            handler() {
                // 设置disabled data
                this.setDisabledData();
            },
            deep: true
        },
        isShowSelectAndMask: {
            handler(val) {
                if (!val) { // 关闭弹框
                    this.roomDetail.dataSource.forEach(x => x.isAllowSelect = false)
                }
            },
            deep: true
        }
    }
}
</script>
<style lang="less">
.mobile-drag {
    width: 100%;
    user-select: none;
    position: relative;

}
.mobile-drag .mobile-drag-info .mobile-drag-info-label {
    text-align: left;
    font-size: 14px;
    color: #0F141A;
    margin-bottom: 8px;
}
.mobile-drag .mobile-drag-info .mobile-drag-info-label span {
    font-size: 14px;
    font-weight: normal;
    color: #333;
}
.mobile-drag .mobile-drag-content {
    position: relative;
    
}
.mobile-drag .mobile-drag-content .mobile-drag-content-slider {
    display: flex;
    align-items: center;
    padding: 0;
}
.mobile-drag .mobile-drag-content .mobile-drag-content-slider li {
    display: flex;
    flex-direction: column;
    flex: 1;
}
.mobile-drag .mobile-drag-content .mobile-drag-content-slider li .block-num {
    height: 22px;
    display: flex;
    justify-content: flex-start;
    position: relative;
    left: -4px;
    color: #A2A4A8;
    font-size: 16px;
    margin-top: 4px;
}
.block-num span {
    position: absolute;
    left: -1px;
    font-size: 12px;
}
.block {
    height: 16px;
    text-align: center;
    box-sizing: border-box;
    border-top: 1px solid #D2D4D6;
    border-bottom: 1px solid #D2D4D6;
    
}
.block:not(.disabledClass):not(.alreadyClass) {
    cursor: pointer;
}
.block.disabledClass {
    background: #cccccc;
}
.block.alreadyClass {
    background: #D3E9FE;
}

.block.integer {
    border-left: 1px solid #D2D4D6;
}
.mobile-drag .mobile-drag-content .mobile-drag-content-slider li:last-child {
    width: 0;
}
.mobile-drag .mobile-drag-content .mobile-drag-content-slider li:last-child .block {
    width: 0 !important;
    background: transparent;
}

.mobile-drag  .mobile-drag-move {
    display: flex;
    position: absolute;
    top: 0;

}
.mobile-drag .mobile-drag-content-tip .mobile-drag-content-tip-triangle {
    width: 0;
    height: 0;
    border-left: 6px solid transparent;
    border-right: 6px solid transparent;
    border-top: 6px solid #b6b2b2;
    margin-left: calc(50% - 6px);
}
.mobile-drag .mobile-drag-mask-small {
    min-height: 100px;
    border-radius: 4px;
    box-shadow: 0 1px 3px 2px rgba(0,0,0,.1);
    position: absolute;
    background: #fff;
    z-index: 10;
}
.mobile-drag .mobile-drag-mask-small .small-title {
    font-size: 14px;
}
.mobile-drag-mask-small .small-date {
    font-size: 12px;
}
.mobile-drag .mobile-drag-mask-small .small-button {
    width: 100%;
}
.mobile-drag-content-item {
    margin-top: 12px;
    background: #fff;
    padding: 12px;
    
}
.content {
    position: relative;
}
.content .header {
    width: 100%;
    text-align: center;
    padding: 4px;
    position: absolute;
    left: 0;
    top: 0;
    background: #fff;
}
.header-tip {
    font-size: 12px;
}
.pop {
    height: 100%;
    overflow: hidden;
    padding-top: 49px;
    padding-bottom: 24px;
    position: relative;
}
.pop-move {
    position: absolute;
    // top: 49px;
    left: 48px;
}
.pop-list {
    padding: 0 12px;
    display: flex;
}
.pop-list-label {
    width: 30px;
    text-align: right;
    margin-right: 6px;
    margin-top: -11px;
}
.pop-list .pop-list-item {
    flex: 1;
    height: 28px;
    border: 1px solid #cccccc;
    border-bottom: 0;
}
.pop-list  .pop-list-item.lastItem {
    border: none;
    border-top: 1px solid #cccccc;
}
.disabledClass {
    background: #cccccc;
}
.alreadyClass {
    background: #D3E9FE;
}
.pop-list  .pop-list-item.allowSelectClass {
    background: red;
    border-top: none;
}
// .allowSelectClass:not(:first-child) {
//     background: blue;
// }
</style>
