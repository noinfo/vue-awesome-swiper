<template>
    <div :class="slideClass">
        <slot></slot>
    </div>
</template>

<script>
    export default {
        name: 'swiper-slide',
        props: {
            onSlideShow: {
                type: Function,
                default: function () {
                }
            },
            onSlideHide: {
                type: Function,
                default: function () {
                }
            },
        },
        data: function () {
            return {
                slideClass: 'swiper-slide',
                slideIndexInParent: -1,
                isActive: false
            }
        },
        ready: function () {
            this.update()
        },
        mounted: function () {
            // determine slide index in parent (swiper)
            this.$parent.$children.forEach((child, index) => {
                if (child === this) {
                    // console.log(index);
                    this.slideIndexInParent = index;
                }
            });
            //console.log('Hello, I am slide ' + this.slideIndexInParent);
            this.update()
            if (this.$parent.options.slideClass) {
                this.slideClass = this.$parent.options.slideClass
            }

            this.$parent.$on('changeActiveSlide', this.onParentChangeActiveSlide);
        },
        destroyed: function () {
            this.$parent.$off('changeActiveSlide', this.onParentChangeActiveSlide);
        },
        updated: function () {
            this.update()
        },
        attached: function () {
            this.update()
        },
        methods: {
            update: function () {
                if (this.$parent && this.$parent.swiper && this.$parent.swiper.update) {
                    this.$parent.swiper.update(true)
                    if (this.$parent.options.loop) {
                        this.$parent.swiper.reLoop()
                    }
                }
            },
            onParentChangeActiveSlide: function (activeSlide) {
                if (activeSlide == this.slideIndexInParent) {
                    //console.log('I became active. I am ' + this.slideIndexInParent);
                    this.isActive = true;
                    if (typeof this.onSlideShow === 'function') {
                        this.onSlideShow();
                    }
                } else {
                    if (this.isActive) {
                        if (typeof this.onSlideHide === 'function') {
                            this.onSlideHide();
                        }
                        this.isActive = false;
                    }


                }
            }
        }
    }
</script>
