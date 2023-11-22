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

        const tooltip = ref();
        const tooltipClassNames = computed(() => {
            return {
                'tooltip show px-2 py-1 rounded-2': true,
                [`bg-${props.theme} text-bg-${props.theme}`]: true,
                ['tooltip-' + props.position]: true
            };
        });

        const isShown = ref(false);

        function show(event) {
            isShown.value = true;

            const rect = event.target.getBoundingClientRect();

            const space = 2;

            function stickToTop(tooltipEl, targetEl) {
                tooltipEl.value.style.top = targetEl.top - tooltipEl.value.clientHeight - space + window.scrollY + 'px';
                tooltipEl.value.style.left = targetEl.left + (targetEl.width / 2) - (tooltipEl.value.clientWidth / 2) + 'px';
            }

            function stickToBottom(tooltipEl, targetEl) {
                tooltipEl.value.style.top = targetEl.bottom + window.scrollY + space + 'px';
                tooltipEl.value.style.left = targetEl.left + (targetEl.width / 2) - (tooltipEl.value.clientWidth / 2) + 'px';
            }

            function stickToStart(tooltipEl, targetEl) {
                tooltipEl.value.style.top = targetEl.top + (targetEl.height / 2) - (tooltipEl.value.clientHeight / 2) + window.scrollY + 'px';

                if (LanguageService.isRtl()) {
                    tooltipEl.value.style.left = targetEl.left + targetEl.width + space + 'px';
                } else {
                    tooltipEl.value.style.left = targetEl.left - tooltipEl.value.clientWidth - space + 'px';
                }
            }

            function stickToEnd(tooltipEl, targetEl) {
                tooltipEl.value.style.top = targetEl.top + (targetEl.height / 2) - (tooltipEl.value.clientHeight / 2) + window.scrollY + 'px';

                if (LanguageService.isRtl()) {
                    tooltipEl.value.style.left = targetEl.left - tooltipEl.value.clientWidth - space + 'px';
                } else {
                    tooltipEl.value.style.left = targetEl.left + targetEl.width + space + 'px';
                }
            }

            function reset() {
                tooltip.value.style.removeProperty('top');
                tooltip.value.style.removeProperty('left');
            }

            nextTick(() => {
                switch (props.position) {
                    case ComponentPosition.TOP:
                        stickToTop(tooltip, rect);
                        break;

                    case ComponentPosition.BOTTOM:
                        stickToBottom(tooltip, rect);
                        break;

                    case ComponentPosition.START:
                        stickToStart(tooltip, rect);
                        break;

                    case ComponentPosition.END:
                        stickToEnd(tooltip, rect);
                        break;
                }

                console.log('top', tooltip.value.style.top)
                console.log('left', tooltip.value.style.left.indexOf('-') > -1)

                if (tooltip.value.style.left.indexOf('-') > -1) {
                    reset();
                    stickToEnd(tooltip, rect);
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
