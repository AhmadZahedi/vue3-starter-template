<template>
<slot name="activator" :on="listeners"></slot>

<Teleport to="body">
    <div
        v-if="isShown"
        :class="tooltipClassNames"
        ref="tooltip"
    >
        <span ref="tooltipArrow" class="arrow"></span>
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
        const tooltipClassNames = computed(() => {
            return [
                'tooltip show px-2 py-1 rounded-2',
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

        function resetTooltipStyles() {
            tooltip.value.style.removeProperty('top');
            tooltip.value.style.removeProperty('left');
        }

        function resetArrowStyles(arrEl) {
            if (!arrEl.style) return;
            arrEl.style.bottom = null;
            arrEl.style.top = null;
            arrEl.style.left = null;
            arrEl.style.transform = null;
            arrEl.style.borderColor = null;
        }

        function stickToTop(tooltipEl, arrowEl, targetRect) {
            tooltipEl.style.top = targetRect.top - tooltipEl.clientHeight - gap + window.scrollY + 'px';
            tooltipEl.style.left = targetRect.left + (targetRect.width / 2) - (tooltipEl.clientWidth / 2) + 'px';

            const tooltipRect = tooltipEl.getBoundingClientRect();

            if (getRectDistanceFrom('left', tooltipRect) < 0) {
                tooltip.value.style.removeProperty('left');
                tooltip.value.style.left = gap + 'px';
            } else if (getRectDistanceFrom('right', tooltipRect) < 0) {
                tooltip.value.style.removeProperty('left');
                tooltip.value.style.right = gap + 'px';
            }

            resetArrowStyles(arrowEl)

            arrowEl.style.bottom = '0';
            arrowEl.style.transform = 'translate(-50%, 100%)';
            arrowEl.style.borderColor = `var(--bs-${props.theme}) transparent transparent transparent`;
            arrowEl.style.left = targetRect.left + (targetRect.width / 2) + (arrowEl.getBoundingClientRect().width / 2) - arrowEl.getBoundingClientRect().left + 'px';
        }

        function stickToBottom(tooltipEl, arrowEl, targetRect) {
            tooltipEl.style.top = targetRect.bottom + window.scrollY + gap + 'px';
            tooltipEl.style.left = targetRect.left + (targetRect.width / 2) - (tooltipEl.clientWidth / 2) + 'px';

            const tooltipRect = tooltipEl.getBoundingClientRect();

            if (getRectDistanceFrom('left', tooltipRect) < 0) {
                tooltip.value.style.removeProperty('left');
                tooltip.value.style.left = gap + 'px';
            } else if (getRectDistanceFrom('right', tooltipRect) < 0) {
                tooltip.value.style.removeProperty('left');
                tooltip.value.style.right = gap + 'px';
            }

            resetArrowStyles(arrowEl)

            arrowEl.style.top = '0';
            arrowEl.style.transform = 'translate(-50%, -100%)';
            arrowEl.style.borderColor = `transparent transparent var(--bs-${props.theme}) transparent`;
            arrowEl.style.left = targetRect.left + (targetRect.width / 2) + (arrowEl.getBoundingClientRect().width / 2) - arrowEl.getBoundingClientRect().left + 'px';
        }

        function stickToStart(tooltipEl, arrowEl, targetRect) {
            tooltipEl.style.top = targetRect.top + (targetRect.height / 2) - (tooltipEl.clientHeight / 2) + window.scrollY + 'px';

            if (LanguageService.isRtl()) {
                tooltipEl.style.left = targetRect.left + targetRect.width + gap + 'px';
            } else {
                tooltipEl.style.left = targetRect.left - tooltipEl.clientWidth - gap + 'px';
            }

            resetArrowStyles(arrowEl)

            arrowEl.style.top = '50%';
            arrowEl.style.transform = 'translate(0%, -50%)';
            arrowEl.style.borderColor = `transparent transparent transparent var(--bs-${props.theme})`;
            arrowEl.style.left = '100%';
        }

        function stickToEnd(tooltipEl, arrowEl, targetRect) {
            tooltipEl.style.top = targetRect.top + (targetRect.height / 2) - (tooltipEl.clientHeight / 2) + window.scrollY + 'px';

            if (LanguageService.isRtl()) {
                tooltipEl.style.left = targetRect.left - tooltipEl.clientWidth - gap + 'px';
            } else {
                tooltipEl.style.left = targetRect.left + targetRect.width + gap + 'px';
            }

            resetArrowStyles(arrowEl)

            arrowEl.style.top = '50%';
            arrowEl.style.transform = 'translate(-100%, -50%)';
            arrowEl.style.borderColor = `transparent var(--bs-${props.theme}) transparent transparent`;
            arrowEl.style.left = '0';
        }

        function show(event) {
            isShown.value = true;

            const rect = event.target.getBoundingClientRect();

            nextTick(() => {
                switch (props.position) {
                    case ComponentPosition.TOP:
                        stickToTop(tooltip.value, tooltipArrow.value, rect);
                        break;

                    case ComponentPosition.BOTTOM:
                        stickToBottom(tooltip.value, tooltipArrow.value, rect);
                        break;

                    case ComponentPosition.START:
                        stickToStart(tooltip.value, tooltipArrow.value, rect);
                        break;

                    case ComponentPosition.END:
                        stickToEnd(tooltip.value, tooltipArrow.value, rect);
                        break;
                }

                const tooltipRect = tooltip.value.getBoundingClientRect();

                if (getRectDistanceFrom('left', tooltipRect) < 0) {
                    resetTooltipStyles();
                    LanguageService.isRtl() ? stickToStart(tooltip.value, tooltipArrow.value, rect) : stickToEnd(tooltip.value, tooltipArrow.value, rect);
                }

                if (getRectDistanceFrom('right', tooltipRect) < 0) {
                    resetTooltipStyles();
                    LanguageService.isRtl() ? stickToEnd(tooltip.value, tooltipArrow.value, rect) : stickToStart(tooltip.value, tooltipArrow.value, rect);
                }

                if (getRectDistanceFrom('top', tooltipRect) < 0) {
                    resetTooltipStyles();
                    stickToBottom(tooltip.value, tooltipArrow.value, rect);
                }

                if (getRectDistanceFrom('bottom', tooltipRect) < 0) {
                    resetTooltipStyles();
                    stickToTop(tooltip.value, tooltipArrow.value, rect);
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
.arrow {
    position: absolute;
    width: 10px;
    height: 10px;
    border: solid 5px;
    transform: translate(-50%, 0);
}
</style>
