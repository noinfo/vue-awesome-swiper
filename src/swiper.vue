<template>
    <div class="swiper-container">
        <slot name="parallax-bg"></slot>
        <div :class="defaultSwiperClasses.wrapperClass">
            <slot></slot>
        </div>
        <slot name="pagination"></slot>
        <slot name="button-prev"></slot>
        <slot name="button-next"></slot>
        <slot name="scrollbar"></slot>
    </div>
</template>

<script>
    var browser = typeof window !== 'undefined'
    if (browser) window.Swiper = require('swiper')
    export default {
        name: 'swiper',
        props: {
            options: {
                type: Object,
                default: function () {
                    return {
                        autoplay: 3500
                    }
                }
            },
            notNextTick: {
                type: Boolean,
                default: function () {
                    return false
                }
            }
        },
        data: function () {
            return {
                defaultSwiperClasses: {
                    wrapperClass: 'swiper-wrapper'
                },
                extendedOptions: {}
            }
        },
        ready: function () {
            this.extendOptions = Object.assign({}, this.options);
            this.extendOptions.onSlideChangeEnd = this.onSlideChangeEnd;

            if (!this.swiper && browser) {
                this.swiper = new Swiper(this.$el, this.extendOptions)
            }
        },
        mounted: function () {
            var self = this;
            this.extendOptions = Object.assign({}, this.options);
            this.extendOptions.onSlideChangeEnd = this.onSlideChangeEnd;

            var mount = function () {
                if (!self.swiper && browser) {
                    delete self.extendOptions.notNextTick
                    var setClassName = false
                    for (var className in self.defaultSwiperClasses) {
                        if (self.defaultSwiperClasses.hasOwnProperty(className)) {
                            if (self.extendOptions[className]) {
                                setClassName = true
                                self.defaultSwiperClasses[className] = self.extendOptions[className]
                            }
                        }
                    }
                    var mountInstance = function () {
                        self.swiper = new Swiper(self.$el, self.extendOptions);
                        self.$emit('changeActiveSlide', self.swiper.realIndex);
                    }
                    setClassName ? self.$nextTick(mountInstance) : mountInstance()
                }
            }
            (this.extendOptions.notNextTick || this.notNextTick) ? mount() : this.$nextTick(mount)
        },
        updated: function () {

            if (this.swiper) {
                this.swiper.update()
            }
        },
        beforeDestroy: function () {
            if (this.swiper) {
                this.swiper.destroy()
                delete this.swiper
            }
        },
        methods: {
            onSlideChangeEnd: function (args) {
                this.$emit('changeActiveSlide', this.swiper.realIndex);
                // call callback from props
                if (this.options.hasOwnProperty('onSlideChangeEnd') && typeof this.options.onSlideChangeEnd === 'function') {
                    if (args) {
                        this.options.onSlideChangeEnd(...args);
                    } else {
                        this.options.onSlideChangeEnd();
                    }

                }
            }
        }
    }
</script>
