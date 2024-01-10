<template>
<slot name="activator" :on="listeners"></slot>

<Teleport to="body">
    <div
        v-if="isShown"
        :class="tooltipClassNames"
        ref="tooltip"
    >
        <span ref="tooltipArrow" class="arrow-test"></span>
        <slot></slot>
    </div>
</Teleport>
</template>

<script>
// Vue
import { computed, nextTick, ref } from 'vue';

// Enums
import ComponentPosition from '@/enums/ComponentPosition';
import ThemeColor from '@/enums/ThemeColor';
import ComponentTrigger from "@/enums/ComponentTrigger";

// Services
import LanguageService from '@/services/language.service';

export default {
    name: 'VTooltip',

    props: {
        position: {
            type: String,
            default: ComponentPosition.TOP,
            validator(value) {
                return Object.values(ComponentPosition).includes(value);
            }
        },
        trigger: {
            type: String,
            default: ComponentTrigger.HOVER,
            validator(value) {
                return Object.values(ComponentTrigger).includes(value);
            }
        },
        theme: {
            type: String,
            default: ThemeColor.PRIMARY,
            validator(value) {
                return Object.values(ThemeColor).includes(value);
            }
        },
        duration: {
            type: Number,
            default: 0
        }
    },

    setup(props) {
        const listeners = computed(() => {
            const result = {};

            switch (props.trigger) {
                case ComponentTrigger.HOVER:
                    result.mouseenter = show;
                    result.mouseleave = hide;
                    break;
                case ComponentTrigger.CLICK:
                    result.click = toggle;
                    break;
                case ComponentTrigger.FOCUS:
                    result.focus = show;
                    result.blur = hide;
                    break;
            }

            return result
        });

        const tooltip = ref(undefined);
        const tooltipArrow = ref(undefined);

        let arrowPosition = ref(undefined);

        const tooltipClassNames = computed(() => {
            return [
                'tooltip show px-2 py-1 rounded-2',
                `arrow-${arrowPosition.value}`,
                `bg-${props.theme}`,
                `text-bg-${props.theme}`,
                `tooltip-${props.position}`
            ];
        });

        const isShown = ref(false);

        const gap = 5;

        function getRectDistanceFrom(bound, rect) {
            switch (bound) {
                case 'top':
                    return rect.top;
                case 'left':
                    return rect.left;
                case 'bottom':
                    return window.innerHeight - rect.bottom;
                case 'right':
                    return window.innerWidth - rect.right;
            }
        }

        function stickToTop(tooltipEl, targetRect) {
            tooltipEl.style.top = targetRect.top - tooltipEl.clientHeight - gap + window.scrollY + 'px';
            tooltipEl.style.left = targetRect.left + (targetRect.width / 2) - (tooltipEl.clientWidth / 2) + 'px';

            arrowPosition.value = 'bottom';

            const tooltipRect = tooltipEl.getBoundingClientRect();

            if (getRectDistanceFrom('left', tooltipRect) < 0) {
                tooltip.value.style.removeProperty('left');
                tooltip.value.style.left = gap + 'px';
            } else if (getRectDistanceFrom('right', tooltipRect) < 0) {
                tooltip.value.style.removeProperty('left');
                tooltip.value.style.right = gap + 'px';
            }
        }

        function stickToBottom(tooltipEl, arrowEl, targetRect) {
            tooltipEl.style.top = targetRect.bottom + window.scrollY + gap + 'px';
            tooltipEl.style.left = targetRect.left + (targetRect.width / 2) - (tooltipEl.clientWidth / 2) + 'px';

            arrowPosition.value = 'top';

            const tooltipRect = tooltipEl.getBoundingClientRect();

            arrowEl.style.borderColor = `transparent transparent var(--bs-${props.theme}) transparent`;
            if (tooltipEl.getBoundingClientRect().left < gap) {
                console.log('1')
                arrowEl.style.left = targetRect.left + (targetRect.width / 2) - gap + 'px';
            } else if (tooltipEl.getBoundingClientRect().left >= gap && tooltipEl.getBoundingClientRect().left + tooltipEl.getBoundingClientRect().width > visualViewport.width - gap) {
                console.log('visualViewport.width - gap', visualViewport.width - gap)
                console.log('tooltipEl.getBoundingClientRect().left + tooltipEl.getBoundingClientRect().width', tooltipEl.getBoundingClientRect().left + tooltipEl.getBoundingClientRect().width)
                arrowEl.style.left = targetRect.left + (targetRect.width / 2) - tooltipEl.getBoundingClientRect().left + 'px';
            } else if (tooltipEl.getBoundingClientRect().left + tooltipEl.getBoundingClientRect().width <= visualViewport.width - gap) {
                console.log('3')
                arrowEl.style.right = targetRect.right + (targetRect.width / 2) - gap + 'px';
            }
            arrowEl.style.top = '0';
            arrowEl.style.transform = 'translate(-50%, -100%)';

            console.log('targetRect.left ', targetRect.left)
            console.log('targetRect.width ', targetRect.width)

            if (getRectDistanceFrom('left', tooltipRect) < 0) {
                tooltip.value.style.removeProperty('left');
                tooltip.value.style.left = gap + 'px';
            } else if (getRectDistanceFrom('right', tooltipRect) < 0) {
                tooltip.value.style.removeProperty('left');
                tooltip.value.style.right = gap + 'px';
            }
        }

        function stickToStart(tooltipEl, targetRect) {
            tooltipEl.style.top = targetRect.top + (targetRect.height / 2) - (tooltipEl.clientHeight / 2) + window.scrollY + 'px';

            if (LanguageService.isRtl()) {
                tooltipEl.style.left = targetRect.left + targetRect.width + gap + 'px';

                arrowPosition.value = 'left';
            } else {
                tooltipEl.style.left = targetRect.left - tooltipEl.clientWidth - gap + 'px';

                arrowPosition.value = 'right';
            }
        }

        function stickToEnd(tooltipEl, targetRect) {
            tooltipEl.style.top = targetRect.top + (targetRect.height / 2) - (tooltipEl.clientHeight / 2) + window.scrollY + 'px';

            if (LanguageService.isRtl()) {
                tooltipEl.style.left = targetRect.left - tooltipEl.clientWidth - gap + 'px';

                arrowPosition.value = 'right';
            } else {
                tooltipEl.style.left = targetRect.left + targetRect.width + gap + 'px';

                arrowPosition.value = 'left';
            }
        }

        function resetStyles() {
            tooltip.value.style.removeProperty('top');
            tooltip.value.style.removeProperty('left');
        }

        function show(event) {
            isShown.value = true;

            const rect = event.target.getBoundingClientRect();

            nextTick(() => {
                switch (props.position) {
                    case ComponentPosition.TOP:
                        stickToTop(tooltip.value, rect);
                        break;

                    case ComponentPosition.BOTTOM:
                        stickToBottom(tooltip.value, tooltipArrow.value, rect);
                        break;

                    case ComponentPosition.START:
                        stickToStart(tooltip.value, rect);
                        break;

                    case ComponentPosition.END:
                        stickToEnd(tooltip.value, rect);
                        break;
                }

                const tooltipRect = tooltip.value.getBoundingClientRect();

                if (getRectDistanceFrom('left', tooltipRect) < 0) {
                    resetStyles();
                    LanguageService.isRtl() ? stickToStart(tooltip.value, rect) : stickToEnd(tooltip.value, rect);
                }

                if (getRectDistanceFrom('right', tooltipRect) < 0) {
                    resetStyles();
                    LanguageService.isRtl() ? stickToEnd(tooltip.value, rect) : stickToStart(tooltip.value, rect);
                }

                if (getRectDistanceFrom('top', tooltipRect) < 0) {
                    resetStyles();
                    stickToBottom(tooltip.value, tooltipArrow.value, rect);
                }

                if (getRectDistanceFrom('bottom', tooltipRect) < 0) {
                    resetStyles();
                    stickToTop(tooltip.value, rect);
                }
            });
        }

        function hide() {
            if (props.duration) {
                setTimeout(() => isShown.value = false, props.duration)
            } else {
                isShown.value = false;
            }
        }

        function toggle(event) {
            if (props.duration) {
                show(event);
                hide();
            } else {
                if (isShown.value === false) {
                    show(event);
                } else {
                    hide();
                }
            }
        }

        return {
            listeners,

            tooltip,
            tooltipArrow,
            tooltipClassNames,

            isShown,
            show,
            hide,
        };
    }

}
</script>

<style scoped>
.arrow-test {
    position: absolute;
    width: 10px;
    height: 10px;
    border: solid 5px;

    transform: translate(-50%, 0);
}

.arrow-bottom::before {
    position: absolute;
    content: '';
    border: solid 5px;
    top: 100%;
    left: 50%;
    transform: translate(-50%, 0);
    border-color: red transparent transparent transparent;
}

.arrow-top::before {
    position: absolute;
    content: '';
    border: solid 5px;
    top: 0;
    left: 50%;
    transform: translate(-50%, -100%);
    border-color: transparent transparent red transparent;
}

.arrow-left::before {
    position: absolute;
    content: '';
    border: solid 5px;
    top: 50%;
    left: 100%;
    transform: translate(0, -50%);
    border-color: transparent transparent transparent red;
}

.arrow-right::before {
    position: absolute;
    content: '';
    border: solid 5px;
    top: 50%;
    left: 0;
    transform: translate(-100%, -50%);
    border-color: transparent red transparent transparent;
}
</style>
