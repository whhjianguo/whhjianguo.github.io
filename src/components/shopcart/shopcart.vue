<template>
    <div>
        <div class="shopcart">
            <div class="content" @click="toggleList">
                <div class="content-left" :class="{'highlight': totalCount > 0}">
                    <div class="logo-wrapper">
                        <div class="logo">
                            <i class="icon-shopping_cart"></i>
                        </div>
                        <div class="num" v-show="totalCount > 0">{{ totalCount }}</div>
                    </div>
                    <div class="price">￥{{ totalPrice }}</div>
                    <div class="desc">赶紧加入购物车吧</div>
                </div>
                <div class="content-right" @click.stop.prevent="pay">
                    <div class="pay" :class="payClass">{{ payDesc }}</div>
                </div>
            </div>
            <div class="ball-container">
                <template v-for="ball in balls">
                    <transition name="drop" @before-enter="beforeEnter" @enter="enter" @after-enter="afterEnter">
                        <div class="ball" v-show="ball.show">
                            <div class="inner inner-hook"></div>
                        </div>
                    </transition>
                </template>
            </div>
            <transition name="fold">
                <div class="shopcart-list" v-show="listShow">
                    <div class="list-header">
                        <h1 class="title">购物车</h1>
                        <span class="empty" @click="empty">清空</span>
                    </div>
                    <div class="list-content" ref="listContent">
                        <ul>
                            <li class="food border-1px" v-for="food in selectFoods">
                                <span class="name">{{ food.name }}</span>
                                <div class="price">￥{{ food.price * food.count }}</div>
                                <div class="cartcontrol-wrapper">
                                    <cartcontrol :food="food" @drop="drop"></cartcontrol>
                                </div>
                            </li>
                        </ul>
                    </div>
                </div>
            </transition>
        </div>
        <transition name="fade">
            <div class="list-mask" v-show="listShow" @click="hideList"></div>
        </transition>
    </div>
</template>

<script type="text/ecmascript-6">
    import BScroll from 'better-scroll';
    import cartcontrol from 'components/cartcontrol/cartcontrol';
    export default{
        props: {
            selectFoods: {
                type: Array,
                default(){
                    return [];
                }
            },
            deliveryPrice: {
                type: Number,
                default: 0
            },
            minPrice: {
                type: Number,
                default: 0
            }
        },
        data() {
            return {
                balls: [
                    {
                        show: false
                    },
                    {
                        show: false
                    },
                    {
                        show: false
                    },
                    {
                        show: false
                    },
                    {
                        show: false
                    }
                ],
                dropBalls: [],
                fold: true /*折叠*/
            };
        },
        computed: {
            totalPrice() {
                let total = 0;
                this.selectFoods.forEach((food) => {
                    total += food.price * food.count;
                });
                return total;
            },
            totalCount() {
                let count = 0;
                this.selectFoods.forEach((food) => {
                    count += food.count;
                });
                return count;
            },
            payDesc() {
                if (this.totalPrice === 0) {
                    return `￥${this.minPrice}元起送`;
                } else if (this.totalPrice < this.minPrice) {
                    let diff = this.minPrice - this.totalPrice;
                    return `还差￥${diff}元起送`;
                } else {
                    return '去结算'
                }
            },
            payClass() {
                if (this.totalPrice < this.minPrice) {
                    return 'not-enough';
                } else {
                    return 'enough';
                }
            },
            listShow() {
                if (!this.totalCount) {
                    this.fold = true;
                    return false;
                }
                let show = !this.fold;
                if (show) {
                    this.$nextTick(() => {
                        if (!this.scroll) {
                            this.scroll = new BScroll(this.$refs.listContent, {
                                click: true
                            });
                        } else {
                            // 重新计算better-scroll高度
                            this.scroll.refresh();
                        }

                    });
                }
                return show;
            }
        },
        methods: {
            drop(el) {
                for (let i = 0; i < this.balls.length; i++) {
                    let ball = this.balls[i];
                    if (!ball.show) {
                        ball.show = true;
                        // 保存el
                        ball.el = el;
                        this.dropBalls.push(ball);
                        return;
                    }
                }
            },
            beforeEnter(el) {
                let count = this.balls.length;
                while (count--) {
                    let ball = this.balls[count];
                    if (ball.show) {
                        // getBoundingClientRect 这个方法返回一个矩形对象，包含四个属性：left、top、right和bottom
                        let rect = ball.el.getBoundingClientRect();
                        let x = rect.left - 32;
                        let y = -(window.innerHeight - rect.top - 22);

                        el.style.display = '';

                        el.style.webkitTransform = `translate3d(0, ${y}px, 0)`;
                        el.style.transform = `translate3d(0, ${y}px, 0)`;

                        let inner = el.getElementsByClassName('inner-hook')[0];
                        inner.style.webkitTransform = `translate3d(${x}px, 0, 0)`;
                        inner.style.transform = `translate3d(${x}px, 0, 0)`;
                    }
                }
            },
            enter(el) {
                /* 触发浏览器重绘 */
                let rf = el.offsetHeight;

                this.$nextTick(() => {
                    el.style.webkitTransform = 'translate3d(0, 0, 0)';
                    el.style.transform = 'translate3d(0, 0, 0)';

                    let inner = el.getElementsByClassName('inner-hook')[0];
                    inner.style.webkitTransform = 'translate3d(0, 0, 0)';
                    inner.style.transform = 'translate3d(0, 0, 0)';
                });
            },
            afterEnter(el) {
                let ball = this.dropBalls.shift();
                if (ball) {
                    ball.show = false;
                    el.style.display = 'none';
                }
            },
            toggleList() {
                if (!this.totalCount) {
                    return;
                }
                this.fold = !this.fold;
            },
            empty() {
                this.selectFoods.forEach((food) => {
                    food.count = 0
                });
            },
            hideList() {
                this.fold = true;
            },
            pay() {
                if(this.totalPrice < this.minPrice){
                    return;
                }
                window.alert(`支付${this.totalPrice}元`);
            }
        },
        components: {
            cartcontrol
        }
    };
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
    @import "../../common/scss/mixins";

    .shopcart {
        position: fixed;
        left: 0;
        bottom: 0;
        z-index: 50;
        width: 100%;
        height: 48px;
        .content {
            display: flex;
            background: #141d27;
            font-size: 0;
            color: rgba(255, 255, 255, .4);
            .content-left {
                flex: 1;
                .logo-wrapper {
                    display: inline-block;
                    vertical-align: top;
                    position: relative;
                    top: -10px;
                    margin: 0 18px;
                    padding: 6px;
                    width: 56px;
                    height: 56px;
                    box-sizing: border-box;
                    border-radius: 50%;
                    background: #141d27;
                    .logo {
                        width: 100%;
                        height: 100%;
                        border-radius: 50%;
                        background: #2b343c;
                        text-align: center;
                        .icon-shopping_cart {
                            font-size: 24px;
                            line-height: 44px;
                            color: #80858a;
                        }
                    }
                    .num {
                        position: absolute;
                        top: 0;
                        right: 0;
                        width: 24px;
                        height: 16px;
                        line-height: 16px;
                        text-align: center;
                        border-radius: 16px;
                        font-size: 9px;
                        font-weight: 700;
                        color: #fff;
                        background: rgb(240, 20, 20);
                        box-shadow: 0 4px 8px 0 rgba(0, 0, 0, .4);
                    }
                }
                .price {
                    vertical-align: top;
                    display: inline-block;
                    line-height: 24px;
                    margin-top: 12px;
                    padding-right: 12px;
                    box-sizing: border-box;
                    border-right: 1px solid rgba(255, 255, 255, .1);
                    font-size: 16px;
                    font-weight: 700;
                }
                .desc {
                    display: inline-block;
                    vertical-align: top;
                    line-height: 24px;
                    margin: 12px 0 0 12px;
                    font-size: 10px;
                }
                &.highlight {
                    .logo-wrapper {
                        .logo {
                            background: rgb(0, 160, 220);
                            .icon-shopping_cart {
                                color: #fff;
                            }
                        }
                    }
                    .price {
                        color: #fff;
                    }
                }
            }
            .content-right {
                flex: 0 0 105px;
                width: 105px;
                .pay {
                    height: 48px;
                    line-height: 48px;
                    text-align: center;
                    font-size: 12px;
                    font-weight: 700;
                    &.not-enough {
                        background: #2b333b;
                    }
                    &.enough {
                        background: #00b43c;
                        color: #fff;
                    }
                }
            }
        }
        .ball-container {
            .ball {
                position: fixed;
                left: 32px;
                bottom: 22px;
                z-index: 200;
                .inner {
                    width: 16px;
                    height: 16px;
                    border-radius: 50%;
                    background: rgb(0, 160, 220);
                }
                &.drop-enter-active {
                    transition: all .4s cubic-bezier(.49, -0.29, .75, .41);
                    .inner {
                        transition: all .4s linear;
                    }
                }
            }
        }
        .shopcart-list {
            position: absolute;
            top: 0;
            left: 0;
            z-index: -1;
            width: 100%;
            transform: translate3d(0, -100%, 0);
            &.fold-enter-active, &.fold-leave-active {
                transition: all .5s;
            }
            &.fold-enter, &.fold-leave-active {
                transform: translate3d(0, 0, 0);
            }
            .list-header {
                height: 40px;
                line-height: 40px;
                padding: 0 18px;
                background: #f3f5f7;
                border-bottom: 1px solid rgba(7, 17, 27, .1);
                .title {
                    float: left;
                    font-size: 14px;
                    color: rgb(7, 17, 27);
                }
                .empty {
                    float: right;
                    font-size: 12px;
                    color: rgb(0, 160, 220);
                }
            }
            .list-content {
                padding: 0 18px;
                max-height: 217px;
                background: #fff;
                overflow: hidden;
                .food {
                    position: relative;
                    padding: 12px 0;
                    box-sizing: border-box;
                    @include border-1px(rgba(7, 17, 27, .1));
                    .name {
                        line-height: 24px;
                        font-size: 14px;
                        color: rgb(7, 17, 27);
                    }
                    .price {
                        position: absolute;
                        right: 90px;
                        bottom: 12px;
                        line-height: 24px;
                        font-weight: 700;
                        font-size: 14px;
                        color: rgb(240, 20, 20);
                    }
                    .cartcontrol-wrapper {
                        position: absolute;
                        bottom: 6px;
                        right: 0;
                    }
                }
            }
        }
    }

    .list-mask {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        z-index: 40;
        background: rgba(7, 17, 27, .6);
        backdrop-filter: blur(10px);
        &.fade-enter-active, .fade-leave-active {
            transition: all .5s;
        }
        .fade-enter, .fade-leave-active {
            opacity: 0
        }
    }
</style>
