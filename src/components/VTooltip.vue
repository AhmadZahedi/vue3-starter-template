<template>
<slot name="activator" :on="listeners"></slot>

<Teleport to="body">
    <div
        v-if="isShown"
        :class="tooltipClassNames"
        ref="tooltip"
    >
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
        const tooltipClassNames = computed(() => {
            return [
                'tooltip show px-2 py-1 rounded-2',
                `bg-${props.theme}`,
                `text-bg-${props.theme}`,
                `tooltip-${props.position}`
            ];
        });

        const isShown = ref(false);

        const gap = 2;

        function stickToTop(tooltipEl, targetEl) {
            tooltipEl.style.top = targetEl.top - tooltipEl.clientHeight - gap + window.scrollY + 'px';
            tooltipEl.style.left = targetEl.left + (targetEl.width / 2) - (tooltipEl.clientWidth / 2) + 'px';
        }

        function stickToBottom(tooltipEl, targetEl) {
            tooltipEl.style.top = targetEl.bottom + window.scrollY + gap + 'px';
            tooltipEl.style.left = targetEl.left + (targetEl.width / 2) - (tooltipEl.clientWidth / 2) + 'px';
        }

        function stickToStart(tooltipEl, targetEl) {
            tooltipEl.style.top = targetEl.top + (targetEl.height / 2) - (tooltipEl.clientHeight / 2) + window.scrollY + 'px';

            if (LanguageService.isRtl()) {
                tooltipEl.style.left = targetEl.left + targetEl.width + gap + 'px';
            } else {
                tooltipEl.style.left = targetEl.left - tooltipEl.clientWidth - gap + 'px';
            }
        }

        function stickToEnd(tooltipEl, targetEl) {
            tooltipEl.style.top = targetEl.top + (targetEl.height / 2) - (tooltipEl.clientHeight / 2) + window.scrollY + 'px';

            if (LanguageService.isRtl()) {
                tooltipEl.style.left = targetEl.left - tooltipEl.clientWidth - gap + 'px';
            } else {
                tooltipEl.style.left = targetEl.left + targetEl.width + gap + 'px';
            }
        }

        function reset() {
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
                        stickToBottom(tooltip.value, rect);
                        break;

                    case ComponentPosition.START:
                        stickToStart(tooltip.value, rect);
                        break;

                    case ComponentPosition.END:
                        stickToEnd(tooltip.value, rect);
                        break;
                }

                const tooltipRect = tooltip.value.getBoundingClientRect();

                function getRectDistanceFrom(bound) {
                    switch (bound) {
                        case 'top':
                            return tooltipRect.top;
                        case 'left':
                            return tooltipRect.left;
                        case 'bottom':
                            return window.innerHeight - tooltipRect.bottom;
                        case 'right':
                            return window.innerWidth - tooltipRect.right;
                    }
                }

                if (getRectDistanceFrom('left') < 0) {
                    reset();
                    LanguageService.isRtl() ? stickToStart(tooltip.value, rect) : stickToEnd(tooltip.value, rect);
                }

                if (getRectDistanceFrom('right') < 0) {
                    reset();
                    LanguageService.isRtl() ? stickToEnd(tooltip.value, rect) : stickToStart(tooltip.value, rect);
                }

                if (getRectDistanceFrom('top') < 0) {
                    reset();
                    stickToBottom(tooltip.value, rect);
                }

                if (getRectDistanceFrom('bottom') < 0) {
                    reset();
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
            tooltipClassNames,

            isShown,
            show,
            hide,
        };
    }

}
</script>
