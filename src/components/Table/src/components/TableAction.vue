<template>
  <div class="tableAction">
    <div class="flex items-center justify-center">
      <template v-for="(action, index) in getActions" :key="`${index}-${action.label}`">
        <n-button v-bind="action" class="mx-2" text size="tiny" type="info">
          {{ action.label }}
          <template #icon v-if="action.hasOwnProperty('icon')">
            <n-icon :component="action.icon" />
          </template>
        </n-button>
      </template>
      <n-dropdown
        v-if="dropDownActions && getDropdownList.length"
        trigger="hover"
        :options="getDropdownList"
        size="small"
        :show-arrow="true"
        @select="select"
      >
        <slot name="more"></slot>
        <n-button
          v-bind="getMoreProps"
          class="mx-2"
          v-if="!$slots.more"
          icon-placement="left"
          quaternary
          type="info"
          size="tiny"
        >
          <n-icon size="14">
            <Dots />
          </n-icon>
        </n-button>
      </n-dropdown>
    </div>
  </div>
</template>

<script lang="ts">
  import { defineComponent, PropType, computed, toRaw } from 'vue';
  import { ActionItem } from '@/components/Table';
  import { usePermission } from '@/hooks/web/usePermission';
  import { isBoolean, isFunction } from '@/utils/is';
  import { Dots } from '@vicons/tabler';
  export default defineComponent({
    name: 'TableAction',
    components: { Dots },
    props: {
      actions: {
        type: Array as PropType<ActionItem[]>,
        default: null,
        required: true,
      },
      dropDownActions: {
        type: Array as PropType<ActionItem[]>,
        default: null,
      },
      style: {
        type: String as PropType<String>,
        default: 'button',
      },
      select: {
        type: Function as PropType<Function>,
        default: () => {},
      },
    },
    setup(props) {
      const { hasPermission } = usePermission();

      const actionType =
        props.style === 'button' ? 'default' : props.style === 'text' ? 'primary' : 'default';
      const actionText =
        props.style === 'button' ? undefined : props.style === 'text' ? true : undefined;

      const getMoreProps = computed(() => {
        return {
          text: actionText,
          type: actionType,
          size: 'small',
        };
      });

      const getDropdownList = computed(() => {
        return (toRaw(props.dropDownActions) || [])
          .filter((action) => {
            return hasPermission(action.auth as string[]) && isIfShow(action);
          })
          .map((action) => {
            const { popConfirm } = action;
            return {
              size: 'small',
              text: actionText,
              type: actionType,
              ...action,
              ...popConfirm,
              onConfirm: popConfirm?.confirm,
              onCancel: popConfirm?.cancel,
            };
          });
      });

      function isIfShow(action: ActionItem): boolean {
        const ifShow = action.ifShow;

        let isIfShow = true;

        if (isBoolean(ifShow)) {
          isIfShow = ifShow;
        }
        if (isFunction(ifShow)) {
          isIfShow = ifShow(action);
        }
        return isIfShow;
      }

      const getActions = computed(() => {
        return (toRaw(props.actions) || [])
          .filter((action) => {
            return hasPermission(action.auth as string[]) && isIfShow(action);
          })
          .map((action) => {
            const { popConfirm } = action;
            //需要展示什么风格，自己修改一下参数
            return {
              size: 'small',
              text: actionText,
              type: actionType,
              ...action,
              ...(popConfirm || {}),
              onConfirm: popConfirm?.confirm,
              onCancel: popConfirm?.cancel,
              enable: !!popConfirm,
            };
          });
      });

      return {
        getActions,
        getDropdownList,
        getMoreProps,
      };
    },
  });
</script>
