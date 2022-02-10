<template>
    <div class="drag" ref="drag">
        
        <div class="drag-content">
            <div class="drag-content-item" v-for="(roomItem, roomInd) in localDisableData" :key="roomInd">
                <div class="drag-info">
                    <p class="drag-info-label">{{roomItem.conference_room_name}}</p>
                </div>
                <ul class="drag-content-slider" :ref="'drag-content-slider' + roomInd" >
                    <li v-for="(item, ind) in roomItem.dataSource" :key="item.localLabel">
                        <div
                            @click="clickStep($event, roomItem, roomInd, item, ind)"
                            @mouseover="blockMouseOver($event, roomItem, roomInd, item, ind)"
                            @mouseleave="blockMouseLeave($event, roomItem, roomInd, item ,ind)"
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
                                        item.alreadyInfo = xx;
                                    }
                                    return flag
                                }),
                            }"
                            :style="{width: blockWidth +'px', height: blockHeight + 'px'}"
                        >
                        </div>
                        <div class="block-num">
                            <span v-show="ind % 2 == 0">{{item.localLabel}}</span>
                        </div>
                    </li>
                </ul>    
            </div>
        </div>
        <!-- 蓝色小块 -->
        <div class="drag-move" ref="drag-move"
            v-show="isShowSelectAndMask"
            :style="{
                left: dLeft + 'px',
                top: dTop + 'px',
                width: dWidth + 'px',
                height: blockHeight + 'px'
            }"
        >
            <div
                class="drag-point left"
                @mousedown="mouseDownL"
                :style="{
                    width: pointW +'px',
                    height: pointH + 'px',
                    left: pointAbsoluteX - pointW + 'px',
                    marginTop: -pointH / 2 + 'px'
                }"
            ></div>
            <div class="drag-move-content"></div>
            <div
                class="drag-point right"
                ref="right"
                @mousedown="mouseDownR"
                :style="{
                    width: pointW+ 'px',
                    height: pointH+ 'px',
                    right: pointAbsoluteX - pointW+ 'px',
                    marginTop: -pointH / 2+ 'px'
                }"
            >
            </div>
        </div>
        <!-- 滑过已预订的展示tip -->
        <div class="drag-content-tip" ref="drag-content-tip"
            @mouseover="contentTipMouseOver"
            @mouseleave="contentTipMouseLeave"
            v-show="showTopTip"
            :style="{
                width: topTipW + 'px',
                left: topTipLeft + 'px',
                top: topTipTop + 'px',
                marginLeft: -topTipW/2 + 'px',
            }"
        >
            <div class="drag-content-tip-text">
                {{alreadyNum}} 已预订 {{alreadyLabelArray.length ? alreadyLabelArray[0] + ' - ' + alreadyLabelArray[1] : ''}}
            </div>
            <div class="drag-content-tip-triangle"></div>
        </div>

        <!-- 点击出现下方框 -->
        <div class="drag-mask-small" ref="drag-mask-small"
            v-show="isShowSelectAndMask"
            :style="{
                width: btmTipW + 'px',
                left: btmTipLeft + 'px',
                top: btmTipTop + 'px'
            }"
        >
            <div class="drag-mask-small-content">
                <p class="small-title">{{roomBtmTipLabel}}</p>
                <p class="small-date">{{fromTimeGetMonthAndDay}} {{dateLabelArray[0]}} - {{dateLabelArray[1]}}</p>
                <button
                    class="small-button"
                    :disabled="!isAllowConfirm"
                    @click="confirm"
                    type="primary"
                >{{isAllowConfirm ? '确定' : '禁用'}}</button>
            </div>
        </div>
    </div>
</template>
<script>
export default {
    name: 'Drag',
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
                            reserveLabel: '18-19',
                            aaa: '随便玩'
                        },
                        {
                            startTime: '1643112000000', // 20:00
                            // startTime: '1642226400000', // 14:00
                            // endTime: '1642228200000', // 14:30
                            // endTime: '1642230000000', // 15:00
                            // endTime: "1642235400000", // 16:30
                            endTime: "1643115600000", // 21:00
                            reserveLabel: '20-21'
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
            dragTop: 0,
            dragLeft: 0,
            dragWidth: '', // 最外层盒子的宽度
            dragTimer: null, // 定时器

            blockWidth: 32, // 每个小块的宽度（非必填）
            blockHeight: 32, // 每个小块的高度（非必填）

            isShowSelectAndMask: false, // 是否展示蓝条和小弹窗
            dLeft: 0, // 蓝条距离父级左侧的距离
            dTop: 0, // 蓝条距离父级上方的距离
            dWidth: this.blockWidth, // 蓝条的宽度 默认传入的块的宽度
            dLeftInd: 0, // 蓝色小块左侧的索引
            dRightInd: 0, // 蓝色小块右侧索引

            pointAbsoluteX: 6, // 小原点水平偏移量
            pointW: 8, // 小圆点的宽度
            pointH: 8, // 小圆点的高度

            showTopTip: false, // 是否展示上方top tip
            topTipW: 200, // 上方top tip的宽度
            topTipTop: 0, // 上方top tip的top
            topTipLeft: 0, // 上方top tip的left

            btmTipW: 250, // 下方确认弹框的宽度
            btmTipLeft: 0, // 下方确认弹框距离左侧的距离
            btmTipTop: 0, // 下方确认弹框距离上侧的距离
            roomBtmTipLabel: '', // 存储下方弹窗的房间label

            isAllowConfirm: false, // 是否允许创建
            dateArray: [], // [8. 9] // 当前选中的区间
            dateLabelArray: [], // [8:00, 9:30] 当前选中的区间
            
            alreadyNum: '', // 已选中的label
            alreadyLabelArray: [], // 已选中的日期存储 [11:30, 14:00]
            

            currentInd: 0, // 当前在第几个
            currentItemInd: 0, // 当前在第几个小块

            // 存储滑过已选中的信息
            alreadyItemInfo: {},

            localDisableData: [] // 当前组件存储禁用和已选中的数据
        }
    },
    methods: {
        contentTipMouseOver() {
            this.showTopTip = true;
            console.log(this.alreadyItemInfo);
        },
        contentTipMouseLeave() {
            this.showTopTip = false;
        },
        blockMouseOver(e, roomItem, roomInd, item, ind) {
            if (!item.alreadyDisalbe) return;
            let haveData = this.localDisableData[roomInd].reserveInfoList.filter(x => !x.isHistory).find(xx => {
                return item.localLabel >= xx.step[0] && item.localLabel < xx.step[1];
            });
            if (haveData) {
                // 展示top tip框
                this.showTopTip = true;

                this.currentInd = roomInd;
                this.currentItemInd = ind;

                // 给top tip的内容赋值
                this.alreadyNum = haveData.reserveLabel || '';
                this.alreadyLabelArray = [this.assemblyTime(haveData.step[0]), this.assemblyTime(haveData.step[1])]
                this.$nextTick(() => {
                    // 设置top tip的位置（left top）
                    let {top: dragTop, left: dragLeft} = this.$refs['drag-content-slider' + roomInd][0].getBoundingClientRect();
                    let {top: dragAllTop} = this.$refs['drag'].getBoundingClientRect();
                    let {height: topTipH} = this.$refs['drag-content-tip'].getBoundingClientRect();
                    
                    this.dragTop = dragTop;
                    this.dragLeft = dragLeft;
                    
                    let startInd = this.getDataSource.findIndex(x => x.localLabel == haveData.step[0]);
                    let endInd = this.getDataSource.findIndex(x => x.localLabel == haveData.step[1]);

                    // 当前步骤条到顶部的距离 - 最外层到顶部的距离 - top tip的高度
                    this.topTipTop = dragTop - dragAllTop - topTipH;
                    this.topTipLeft = (endInd - startInd) * this.blockWidth / 2 + startInd * this.blockWidth;
                })

                // 存储滑过已选中的信息
                this.alreadyItemInfo = this.localDisableData[this.currentInd].dataSource[this.currentItemInd]['alreadyInfo'];
                console.log(this.alreadyItemInfo);
            }
        },
        blockMouseLeave() {
            this.showTopTip = false;
        },
        confirm() { // 小弹框的确定按钮
            if (!this.isAllowConfirm) return;
            let {year, mon, date} = this.getAssemblyDate(this.currentDate);
            
            this.$emit('confirm', {
                dateParams: {
                    startTime: new Date(year+'/'+mon+'/'+date+' '+this.dateLabelArray[0]).getTime(),
                    startDate: this.dateArray[0],
                    startLabel: this.dateLabelArray[0],
                    endTime: new Date(year+'/'+mon+'/'+date+' '+this.dateLabelArray[1]).getTime(),
                    endDate: this.dateArray[1],
                    endLabel: this.dateLabelArray[1]
                },
                roomInfo: this.roomList[this.currentInd],
                currentDate: this.currentDate,
                compareData: this.compareData
            })
        },
        reset() { // 容器点击 清除
            this.isShowSelectAndMask = false;
            this.dLeft = 0;
            this.dWidth = this.blockWidth;
            this.showTopTip = false;
            this.btmTipLeft = 0;
            this.alreadyItemInfo = {};
        },
        clickStep(e, roomItem, roomInd, item, ind) { // 步骤块点击
            let cName = e.target.className;
            // 如果点击的是过去时间 或者 已选过的时间，则return
            if (cName.indexOf('disabledClass') != -1 || cName.indexOf('alreadyClass') != -1) return;

            // 存储蓝色小块的左右索引
            this.dLeftInd = ind;
            this.dRightInd = ind + 1;

            this.currentInd = roomInd;

            // 设置下方弹框的label
            this.roomBtmTipLabel = roomItem.conference_room_name || '';

            let {left: dragLeft, top: dragTop, width: dragSlierW} = this.$refs['drag-content-slider' + this.currentInd][0].getBoundingClientRect();
            let {top: dragAllTop} = this.$refs['drag'].getBoundingClientRect();
            this.dragLeft = dragLeft;
            this.dragTop = dragTop;
            
            this.isShowSelectAndMask = true;
            
            // 设置蓝色小块的距离
            this.dLeft = ind * this.blockWidth;
            this.dTop = dragTop - dragAllTop;

            // 设置下方弹框的距离 (left top)
            // 如果下方的弹框超出右侧的距离，则靠右对齐，否则和蓝色小块对齐
            this.btmTipLeft = this.dLeft + this.btmTipW >= dragSlierW ? dragSlierW - this.btmTipW : this.dLeft;
            this.btmTipTop =  this.dTop + this.blockHeight + 20;

            // 重置蓝色小块的宽度
            this.dWidth = this.blockWidth;

            // 获取当前左右左边的位置
            this.getCurrentStep();
        },
        mouseDownL(e) {
            // 当前鼠标按下的位置
            let disX = e.clientX;

            let domL = this.dLeft || 0;

            let {width: dragSlierW, left: dragLeft} = this.$refs['drag-content-slider' + this.currentInd][0].getBoundingClientRect();

            // 右边点到父级左侧的距离
            let rightL = this.$refs['right'].getBoundingClientRect().left - dragLeft;
            document.onmousemove = (eM) => {
                // 要移动的距离（左边点到父级左侧的距离）
                let l = domL + (Math.floor((eM.clientX - disX) / this.blockWidth)) * this.blockWidth;
                // 边界条件：左侧
                l = l <= 0 ? 0 : l;

                // 最小宽度this.blockWidth限制
                this.dWidth = rightL - l + this.pointAbsoluteX <= this.blockWidth ? this.blockWidth : rightL - l + this.pointAbsoluteX;

                // 左侧点到最小宽度后，不能继续向右移动
                if (rightL - l + this.pointAbsoluteX < this.blockWidth) return;

                // 设置下方弹框的距离
                this.btmTipLeft = l + this.btmTipW >= dragSlierW ? dragSlierW - this.btmTipW : l;

                // 设置蓝色小块的距离
                this.dLeft = l;

                // 获取当前左右左边的位置
                this.getCurrentStep('setdLeftInd');
            }
            document.onmouseup = () => {
                document.onmousemove = null;
                document.onmouseup = null;
                
            }
        },
        mouseDownR(e) {
            // 获取鼠标按下的位置
            let disX = e.clientX;

            let domW = this.dWidth || 0;

            let {left: dragLeft, width: dragSlierW} = this.$refs['drag-content-slider' + this.currentInd][0].getBoundingClientRect();

            // 右侧边界
            let rightBorder = dragLeft + dragSlierW;

            document.onmousemove = (eM) => {
                // 新增的宽度 = 移动的距离 - 鼠标按下的距离
                // dom总宽度 = 原来的宽度 + 新增的宽度
                let w = domW + (Math.floor((eM.clientX - disX) / this.blockWidth)) * this.blockWidth;

                // 最小宽度this.blockWidth
                w = w > this.blockWidth ? w : this.blockWidth;

                // 最大宽度限制父级的宽度
                if (eM.clientX > rightBorder) return;
                
                this.dWidth = w;

                // 获取当前左右左边的位置
                this.getCurrentStep('setdRightInd');
            }
            document.onmouseup = (eU) => {
                // eU.stopPropagation();
                if (eU.clientX == disX) return;
                document.onmousemove = null;
                document.onmouseup = null;
            }
        },

        // 获取当前左右左边的位置
        getCurrentStep(str) {
            if (str == 'setdRightInd') {
                this.dRightInd = Number(this.dLeftInd * 1 + this.dWidth / this.blockWidth).toFixed();
            } else if (str == 'setdLeftInd') {
                this.dLeftInd = Number(this.dRightInd - this.dWidth / this.blockWidth).toFixed()
            }
            let l = this.localDisableData[this.currentInd].dataSource[this.dLeftInd]['localLabel'];
            let r = this.localDisableData[this.currentInd].dataSource[this.dRightInd]['localLabel'];
            this.dateArray = [l, r];

            // 判断是否允许创建
            this.allowConfirm(this.dLeftInd, this.dRightInd);
            // 设置小弹框的时间
            this.dateLabelArray = [this.assemblyTime(l), this.assemblyTime(r)];
        },
        
        // 判断是否允许创建
        allowConfirm(lInd, rInd) {
            let disableLen = this.localDisableData[this.currentInd].dataSource.slice(lInd, rInd).filter(x => x.isDisabed);
            this.isAllowConfirm = !disableLen.length;
        },

        // 设置disabled data
        setDisabledData() {
            this.localDisableData = JSON.parse(JSON.stringify(this.roomList));
            // 获取当前年月日的时间戳以及时、分
            let {ymdTime: currentDate, hour, min} = this.getAssemblyDate();

            // 获取传入的年月日的时间戳
            let {ymdTime: propDateTime} = this.getAssemblyDate(this.currentDate);

            this.localDisableData.forEach(x => {
                x.reserveInfoList && x.reserveInfoList.forEach(xx => {
                    let {hour: startH, min: startM} = this.getAssemblyDate(xx.startTime * 1);
                    let {hour: endH, min: endM} = this.getAssemblyDate(xx.endTime * 1);
                    let startTime = startM >= 30 ? startH + '.5' : startH;
                    let endTime = endM == 0 ? endH : endM <= 30 ? endH + 0.5 : endH * 1 + 1;
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

        // 动态计算每个小块的宽高
        calculateDragBlockWidth() {
            this.dragWidth = this.$refs['drag'].clientWidth || 1000;
            this.blockWidth = Math.floor((this.dragWidth - 20) / (this.getDataSource.length - 1));
            this.reset();
        }
    },
    computed: {
        getDataSource() { // 转换数据源 把 [1, 2] 转成 [{localLabel: 1}, {localLabel: 2}]
            return this.dataSource.map(x => {
                return {
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
        // // 动态计算小块的宽高
        this.calculateDragBlockWidth();
        this.dragTimer = null;
        window.onresize = () => {
            if (this.dragTimer) clearTimeout(this.dragTimer);
            this.dragTimer = setTimeout(() => {
                this.calculateDragBlockWidth();
            }, 100);
        }

        // 设置disabled data
        this.setDisabledData();

        window.addEventListener('click', (e) => {
            // 非当前元素
            if(!e.target.closest(".drag")) {
                this.reset();
            }
        })
        // }, {passive: true})
        
    },
    watch: {
        roomList: {
            handler() {
                // 设置disabled data
                this.setDisabledData();

                // 重置
                this.reset();
            },
            deep: true
        }
    }
}
</script>
<style lang="less">
.drag {
    width: 100%;
    user-select: none;
    position: relative;

}
.drag .drag-info .drag-info-label {
    text-align: left;
    font-size: 16px;
    font-weight: bold;
    color: #0F141A;
    margin-bottom: 8px;
}
.drag .drag-info .drag-info-label span {
    font-size: 14px;
    font-weight: normal;
    color: #333;
}
.drag .drag-content {
    position: relative;
    
}
.drag .drag-content .drag-content-slider {
    display: flex;
    align-items: center;
    width: fit-content;
    padding: 0;
}
.drag .drag-content .drag-content-slider li {
    display: flex;
    flex-direction: column;
   
}
.drag .drag-content .drag-content-slider li .block-num {
    height: 22px;
    display: flex;
    justify-content: flex-start;
    position: relative;
    left: -4px;
    color: #A2A4A8;
    font-size: 12px;
    margin-top: 4px;
}
.block {
    // background: #ccc;
    text-align: center;
    box-sizing: border-box;
    border-top: 1px solid #D2D4D6;
    border-bottom: 1px solid #D2D4D6;
    
}
.block:not(.disabledClass):not(.alreadyClass) {
    cursor: pointer;
}
.block.disabledClass {
    background: #F3F4F4;
}
.block.alreadyClass {
    background: #D3E9FE;
}

.block.integer {
    border-left: 1px solid #D2D4D6;
}
.block.odd {
    border-left: 1px solid #E8EAEA;
}
.drag .drag-content .drag-content-slider li:last-child {
    width: 0;
}
.drag .drag-content .drag-content-slider li:last-child .block {
    width: 0 !important;
    background: transparent;
}

.drag  .drag-move {
    display: flex;
    position: absolute;
    top: 0;

}
.drag .drag-move .drag-point {
    background: #5199F4;
    cursor: col-resize;
    position: absolute;
    top: 50%;
    border-radius: 50%;
    z-index: 1;
    
}
.drag .drag-move .drag-point &.right {
    left: unset;
} 
.drag .drag-move .drag-move-content {
    width: 100%;
    height: 100%;
    background: #D3E9FE;
    cursor: pointer;
    opacity: 0.8;
}


.drag .drag-content-tip {
    position: absolute;
    line-height: 24px;
    // min-height: 40px;
    height: fit-content;
}
.drag .drag-content-tip .drag-content-tip-text {
    z-index: 6;
    text-align: center;
    width: 100%;
    background: #fff;
    border-radius: 4px;
    box-shadow: 0 1px 3px 2px rgba(0,0,0,.1);
    font-size: 12px;
    font-weight: bold;
    padding: 5px;
}
.drag .drag-content-tip .drag-content-tip-triangle {
    width: 0;
    height: 0;
    border-left: 6px solid transparent;
    border-right: 6px solid transparent;
    border-top: 6px solid #b6b2b2;
    // position: absolute;
    // left: 50%;
    // margin-left: -6px;
    margin-left: calc(50% - 6px);
}
.drag .drag-mask-small {
    min-height: 100px;
    border-radius: 4px;
    box-shadow: 0 1px 3px 2px rgba(0,0,0,.1);
    position: absolute;
    background: #fff;
    z-index: 10;
}
.drag .drag-mask-small .drag-mask-small-content {
    padding: 10px;
}
.drag .drag-mask-small .small-title {
    font-size: 14px;
}
.drag-mask-small .small-date {
    font-size: 12px;
}
.drag .drag-mask-small .small-button {
    width: 100%;
}
</style>
