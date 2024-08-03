<template>
  <div class="multi-thumb-slider">
    <div id="tag-slider"
         class="tag-slider"
         style="width: 100%;
                    display: flex;
                    background-color: transparent;">
      <div
          class="tag"
          :class="{'change-position' : widths[index] <= 30}"
          v-for="(tag, index) in sliderData.tags"
          :key="index"
          :style="[
                     styles.tag,
                     {background: tag.color},
                     {width: widths[index]+ '%' }
                     ]">
        <div class="tag-inner">
          <span>{{ `${widths[index]}%` }}</span>
        </div>
        <div class="tag-info">
          <span class="tag-name">{{ `${tag.name}` }}</span>
          <span v-if="totalBudget">{{ `\u00A0- \u00A0` }}</span>
          <!--             Don't remove / Check first performance for a while-->
          <!--            :class="{'text-danger': findPlatformForItsIndex(index, tag.minValue).show && error.has}"-->
          <span
              class="tag-percent"
              v-if="totalBudget"
              :class="{'text-danger': tag.error}"
          >
            {{ `${( widths[index] / 100 * totalBudget).toLocaleString()}$` }}
          </span>
        </div>
        <div
            :style="styles.sliderButton"
            @pointerdown="onSliderSelect({index: index,
            tag: tag,
            disabled: tag.disabled },
            $event)"
            class="slider-button"
            :class="{'disabled':
            tag.disabled }"
        >
        </div>
      </div>
    </div>
    <div class="error mt-5" v-if="error.has">
      <p v-for="(message, index) in error.messages" :key="index">{{ message }}</p>
    </div>
    <a class="set-default" :class="{'extra-margin': !error.has}" v-if="!hideSetAsDefault && showResetBtn && !isEdit" @click="resetToDefault">
<!--      <img src="@/assets/icons/commonIcons/reset-icon.svg"-->
<!--           :alt="'Goal-Icon'" >-->
      <span>{{ 'reset' }}</span>
    </a>
    <div class="mb-5" v-else/>
  </div>
</template>
<script>
export default {
    name: 'multi-thumb-slider',
    data() {
        return {
            styles: {
                tag: {
                    padding: '0',
                    textAlign: 'center',
                    position: 'relative',
                    borderRightWidth: '1px',
                    borderRightStyle: 'solid',
                    borderRightColor: '#fff',
                    boxSizing: 'border-box',
                    borderLeftWidth: '1px',
                    borderLeftStyle: 'solid',
                    borderLeftColor: '#fff'
                },
                sliderButton: {
                    width: '16px',
                    height: '16px',
                    backgroundColor: 'white',
                    position: 'absolute',
                    border: '1px solid #494949',
                    borderRadius: '2em',
                    right: 'calc(-9px)',
                    top: 0,
                    display: 'flex',
                    justifyContent: 'center',
                    alignItems: 'center',
                    bottom: 0,
                    margin: 'auto',
                    zIndex: 10,
                    cursor: 'ew-resize',
                    userSelect: 'none'
                }
            }
        };
    },
    props: {
        options: {
            type: Array,
            default: () => []
        },
        hideSetAsDefault: Boolean,
        totalBudget: {
            type: Number,
            default: 0
        },
        error: {
            type: Object,
            default: () => {}
        },
        isEdit: {
            type: Boolean,
            default: false
        }
    },
    computed: {
        showResetBtn() {
            if (this.widths.length !== this.sliderData.defaults.length) {
                return false;
            }
            for (let i = 0; i < this.widths.length; i++) {
                if (this.widths[i] !== this.sliderData.defaults[i]) {
                    return true;
                }
            }
            return false;
        },
        sliderData() {
            let modelValues = [],
                keys = [],
                tags = [],
                defaults = [];

            this.options.forEach(portion => {
                if (portion.key in this.$attrs) {
                    modelValues.push(this.$attrs[portion.key]);
                    keys.push(portion.key);
                    tags.push(
                        {
                            name: portion.label,
                            color: portion.color,
                            disabled: portion.disabled,
                            error: portion.error
                        }
                    );
                    defaults.push(portion.default);
                }
            });

            return {
                modelValues,
                keys,
                tags,
                defaults
            };
        },
        widths: {
            get() {
                if (this.sliderData.modelValues.length) {
                    return this.sliderData.modelValues;
                } else {
                    return this.sliderData.defaults;
                }
            },
            set(widths) {
                widths.forEach((value, index) => {
                    this.$emit(`update:${this.sliderData.keys[index]}`, value);
                });
            }
        }
    },
    methods: {
        getPercentage(containerWidth, distanceMoved) {
            return (distanceMoved / containerWidth) * 100;
        },
        limitNumberWithinRange(value, min, max) {
            return Math.min(Math.max(value, min), max);
        },
        nearestN(N, number) {
            return Math.ceil(number / N) * N;
        },
        onSliderSelect(parametersFromSliderBtn, e) {
            e.preventDefault();
            if (!parametersFromSliderBtn.disabled) {
                let widths = this.widths;


                document.body.style.cursor = 'ew-resize';
                const startDragX = e.pageX;
                const sliderWidth = document.getElementById('tag-slider').offsetWidth;

                const resize = (e) => {
                    e.preventDefault();
                    const endDragX = e.touches ? e.touches[0].pageX : e.pageX;

                    const distanceMoved = endDragX - startDragX;
                    const maxPercent = widths[parametersFromSliderBtn.index] + widths[parametersFromSliderBtn.index + 1];

                    const percentageMoved = this.nearestN(1,
                        this.getPercentage(sliderWidth, distanceMoved));

                    const _widths = widths.slice();

                    const prevPercentage = _widths[parametersFromSliderBtn.index];

                    const newPercentage = prevPercentage + percentageMoved;
                    const currentSectionWidth = this.limitNumberWithinRange(
                        newPercentage,
                        0,
                        maxPercent
                    );
                    _widths[parametersFromSliderBtn.index] = currentSectionWidth;

                    const nextSectionIndex = parametersFromSliderBtn.index + 1;

                    const nextSectionNewPercentage = _widths[nextSectionIndex] - percentageMoved;
                    const nextSectionWidth = this.limitNumberWithinRange(
                        nextSectionNewPercentage,
                        0,
                        maxPercent
                    );
                    _widths[nextSectionIndex] = nextSectionWidth;


                    this.widths = _widths;
                };

                window.addEventListener('pointermove', resize);
                window.addEventListener('touchmove', resize);

                const removeEventListener = () => {
                    window.removeEventListener('pointermove', resize);
                    window.removeEventListener('touchmove', resize);
                };

                const handleEventUp = (e) => {
                    e.preventDefault();
                    document.body.style.cursor = 'initial';
                    removeEventListener();
                };

                window.addEventListener('touchend', handleEventUp);
                window.addEventListener('pointerup', handleEventUp);


                this.color = parametersFromSliderBtn.tag.color;
            }
        },
        resetToDefault() {
            this.widths = this.sliderData.defaults;
        },
        // Don't remove / Check first performance for a while
        // findPlatformForItsIndex(index, minimumAmount) {
        //     // console.log('minimumAmount', minimumAmount);
        //     // const minimumAmounts = [20, 30, 20, 30];
        //     const amount = (this.widths[index] / 100) * this.totalBudget;
        //     // const minimumAmount = minimumAmounts[index];
        //     const show = (!this.isEdit && this.widths[index] !== 0) ? false : amount < minimumAmount;
        //
        //     return { minimumAmount, show };
        // }
    }
};

</script>
<style lang="scss" scoped>
.multi-thumb-slider {
  .tag:first-of-type {
    border-radius: 30px 0 0 30px;
    border-left: none !important;

    .tag-info {
      position: absolute !important;
      bottom: -30px !important;
      left: 50% !important;
      transform: translateX(-50%) !important;
      top: initial !important;
    }

  }
  .tag:last-of-type {
    border-radius: 0 30px 30px 0;
    border-right: none !important;
    .tag-info {
      position: absolute !important;
      bottom: -30px !important;
      left: 50% !important;
      transform: translateX(-50%) !important;
      top: initial !important;
    }
    &:first-child{
      border-radius: 30px !important;
    }
  }

  .tag.slider-button.disabled {
    cursor: not-allowed !important;
  }
  .tag:last-of-type>.slider-button {
    display:none !important;
  }
  .tag.change-position {
    .tag-info {
      top: -26px;
      bottom: initial;
    }
  }

  .tag {
    height: 24px;

    .tag-inner {
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      span {
        color: #fff;
        font-size: 12px;
        font-family: sans-serif;
      }
    }

    .tag-info {
      width: 140px;
      position: absolute;
      bottom: -30px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      align-items: center;
      justify-content: center;
      font-family: sans-serif;
      background-color: #fff;
      border-radius: 5px;
      flex-wrap: wrap;

      .tag-name {
        color: #000;
        font-size: 12px;
        font-family: sans-serif;
        font-style: normal;
        font-weight: 400;
        line-height: 16px;
        text-align: center;
      }
      .tag-percent {
        color: #0873F6;
        font-family: sans-serif;
        font-style: normal;
        font-size: 12px;
        line-height: 20px;
        text-align: center;
      }
    }
  }

  .set-default.extra-margin {
    margin-top: 60px;
  }
  .set-default {
    width: 70px;

    img {
      margin-right: 8px;
    }

    span {
      margin-top: 2px;
    }

    height: 24px;
    display: flex;
    align-items: center;
    margin-top: 20px;
    cursor: pointer;
    text-decoration: none;
    font-style: normal;
    font-weight: 400;
    font-size: 14px;
    color: #000;
    font-family: sans-serif;
  }

  .error {
    font-size: 12px;
    font-family: sans-serif;
  }
}
</style>
