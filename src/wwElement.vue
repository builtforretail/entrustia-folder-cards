<template>
  <div class="folder-card-list" :style="containerStyle">

    <div
      v-for="item in processedItems"
      :key="item.id"
      class="folder-card"
      :style="cardStyle"
    >
      <!-- Action Row -->
      <div class="card-actions">
        <button
          class="btn-action btn-open"
          :style="openButtonStyle"
          type="button"
          @click="handleOpen(item)"
        >
          Open
        </button>
        <button
          class="btn-action btn-edit"
          :style="editButtonStyle"
          type="button"
          @click="handleEdit(item)"
        >
          Edit
        </button>
      </div>

      <!-- Folder Name -->
      <div class="card-field folder-name-field">
        <span
          class="folder-name"
          :style="folderNameStyle"
          role="button"
          tabindex="0"
          @click="handleNameClick(item)"
          @keydown.enter="handleNameClick(item)"
          @keydown.space.prevent="handleNameClick(item)"
        >
          {{ item.name }}
        </span>
      </div>

      <!-- Files -->
      <div class="card-field">
        <span class="field-label" :style="labelStyle">Files</span>
        <span class="field-value" :style="valueStyle">{{ item.file_count ?? 0 }}</span>
      </div>

      <!-- AI Policy -->
      <div class="card-field">
        <span class="field-label" :style="labelStyle">AI Policy</span>
        <span class="field-value ai-policy-value" :style="valueStyle">
          <span v-if="getAiPolicyIcon(item.read_content_mode)" class="ai-policy-icon" aria-hidden="true">{{ getAiPolicyIcon(item.read_content_mode) }}</span>
          {{ getAiPolicyText(item.read_content_mode) }}
        </span>
      </div>

      <!-- Active Public Page -->
      <div class="card-field">
        <span class="field-label" :style="labelStyle">Active Public Page</span>
        <span class="field-value checkbox-value">
          <span
            class="checkbox-box"
            :style="item.has_public_portal ? checkedBoxStyle : uncheckedBoxStyle"
            aria-hidden="true"
          >
            <svg v-if="item.has_public_portal" width="10" height="8" viewBox="0 0 10 8" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M1 4L3.5 6.5L9 1" stroke="white" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </span>
        </span>
      </div>

      <!-- View Only Link -->
      <div class="card-field">
        <span class="field-label" :style="labelStyle">View Only Link</span>
        <button
          class="btn-share-link"
          :style="shareLinkButtonStyle"
          type="button"
          @click="handleShareLink(item)"
          @mouseenter="setShareHover(item.id, true)"
          @mouseleave="setShareHover(item.id, false)"
          title="Copy view-only link"
        >
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <rect x="8" y="2" width="8" height="6" rx="1" stroke="currentColor" stroke-width="1.8" stroke-linejoin="round"/>
            <rect x="8" y="16" width="8" height="6" rx="1" stroke="currentColor" stroke-width="1.8" stroke-linejoin="round"/>
            <rect x="2" y="9" width="8" height="6" rx="1" stroke="currentColor" stroke-width="1.8" stroke-linejoin="round"/>
            <line x1="16" y1="5" x2="20" y2="5" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
            <line x1="20" y1="5" x2="16" y2="12" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
            <line x1="16" y1="19" x2="20" y2="19" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
            <line x1="20" y1="19" x2="16" y2="12" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
          </svg>
        </button>
      </div>
    </div>

    <!-- Empty state -->
    <div v-if="!processedItems.length" class="empty-state" :style="emptyStateStyle">
      No folders to display.
    </div>
  </div>
</template>

<script>
import { computed, ref, watch } from 'vue';

export default {
  name: 'FolderCardList',

  props: {
    uid: { type: String, required: true },
    content: { type: Object, required: true },
    /* wwEditor:start */
    wwEditorState: { type: Object, required: true },
    /* wwEditor:end */
  },

  emits: ['trigger-event'],

  setup(props, { emit }) {
    /* wwEditor:start */
    const isEditing = computed(() => props.wwEditorState?.isEditing);
    /* wwEditor:end */

    // ── Call composables at top level of setup() — NEVER inside computed ──
    const { resolveMappingFormula } = wwLib.wwFormula.useFormula();

    const { value: selectedItem, setValue: setSelectedItem } =
      wwLib.wwVariable.useComponentVariable({
        uid: props.uid,
        name: 'selectedItem',
        type: 'object',
        defaultValue: null,
      });

    const { value: itemCount, setValue: setItemCount } =
      wwLib.wwVariable.useComponentVariable({
        uid: props.uid,
        name: 'itemCount',
        type: 'number',
        defaultValue: 0,
      });

    // Hover state for share link button
    const shareHoverState = ref({});
    const setShareHover = (id, val) => {
      shareHoverState.value = { ...shareHoverState.value, [id]: val };
    };

    const processedItems = computed(() => {
      const raw = props.content?.data;
      const items = Array.isArray(raw) ? raw : [];

      return items.map((item) => {
        if (!item || typeof item !== 'object') return null;
        const id = resolveMappingFormula(props.content?.dataIdFormula, item) ?? item?.id;
        const name = resolveMappingFormula(props.content?.dataNameFormula, item) ?? item?.name;
        const file_count = resolveMappingFormula(props.content?.dataFileCountFormula, item) ?? item?.file_count;
        const read_content_mode = resolveMappingFormula(props.content?.dataReadContentModeFormula, item) ?? item?.read_content_mode;
        const has_public_portal = resolveMappingFormula(props.content?.dataHasPublicPortalFormula, item) ?? item?.has_public_portal;

        return {
          ...item,
          id: id ?? 'item-' + Math.random(),
          name: name ?? 'Untitled',
          file_count: file_count ?? 0,
          read_content_mode: read_content_mode ?? '',
          has_public_portal: Boolean(has_public_portal),
          _original: item,
        };
      }).filter(Boolean);
    });

    watch(processedItems, (items) => { setItemCount(items?.length ?? 0); }, { immediate: true });

    const resolvedPrimaryColor = computed(() => props.content?.primaryColor || '#2d6a4f');
    const resolvedOutlineColor = computed(() => props.content?.outlineColor || '#2d6a4f');

    const containerStyle = computed(() => ({
      '--fcl-primary': resolvedPrimaryColor.value,
      '--fcl-outline': resolvedOutlineColor.value,
      '--fcl-card-bg': props.content?.cardBackground || '#ffffff',
      '--fcl-card-border': props.content?.cardBorderColor || '#e5e7eb',
      '--fcl-card-radius': (props.content?.cardBorderRadius ?? 8) + 'px',
      '--fcl-label-color': props.content?.labelTextColor || '#6b7280',
      '--fcl-value-color': props.content?.valueTextColor || '#111827',
      '--fcl-name-color': props.content?.folderNameColor || '#2d6a4f',
      '--fcl-gap': (props.content?.cardGap ?? 12) + 'px',
      '--fcl-font-size': (props.content?.fontSize ?? 14) + 'px',
      display: 'flex',
      flexDirection: 'column',
      gap: (props.content?.cardGap ?? 12) + 'px',
      width: '100%',
    }));

    const cardStyle = computed(() => ({
      background: props.content?.cardBackground || '#ffffff',
      border: '1px solid ' + (props.content?.cardBorderColor || '#e5e7eb'),
      borderRadius: (props.content?.cardBorderRadius ?? 8) + 'px',
      fontSize: (props.content?.fontSize ?? 14) + 'px',
    }));

    const openButtonStyle = computed(() => ({
      backgroundColor: resolvedPrimaryColor.value,
      color: '#ffffff',
      borderColor: resolvedPrimaryColor.value,
      fontSize: (props.content?.fontSize ?? 14) + 'px',
    }));

    const editButtonStyle = computed(() => ({
      backgroundColor: '#ffffff',
      color: resolvedOutlineColor.value,
      borderColor: resolvedOutlineColor.value,
      fontSize: (props.content?.fontSize ?? 14) + 'px',
    }));

    const shareLinkButtonStyle = computed(() => ({
      color: resolvedPrimaryColor.value,
      borderColor: resolvedPrimaryColor.value,
    }));

    const folderNameStyle = computed(() => ({
      color: props.content?.folderNameColor || '#2d6a4f',
      fontSize: (props.content?.fontSize ?? 14) + 'px',
    }));

    const labelStyle = computed(() => ({
      color: props.content?.labelTextColor || '#6b7280',
      fontSize: (props.content?.fontSize ?? 14) + 'px',
    }));

    const valueStyle = computed(() => ({
      color: props.content?.valueTextColor || '#111827',
      fontSize: (props.content?.fontSize ?? 14) + 'px',
    }));

    const checkedBoxStyle = computed(() => ({
      backgroundColor: resolvedPrimaryColor.value,
      borderColor: resolvedPrimaryColor.value,
    }));

    const uncheckedBoxStyle = computed(() => ({
      backgroundColor: '#ffffff',
      borderColor: '#d1d5db',
    }));

    const emptyStateStyle = computed(() => ({
      color: props.content?.labelTextColor || '#6b7280',
      fontSize: (props.content?.fontSize ?? 14) + 'px',
    }));

    const getAiPolicyText = (mode) => {
      const m = String(mode ?? '').trim();
      if (m === 'Enabled') return 'Deep scan (content analysis)';
      if (m === 'Metadata') return 'Quick scan (metadata only)';
      return 'Disabled';
    };

    const getAiPolicyIcon = (mode) => {
      const m = String(mode ?? '').trim();
      if (m === 'Enabled') return '🔍';
      if (m === 'Metadata') return '⚡';
      return '';
    };

    const handleOpen = (item) => {
      const payload = item?._original ?? item;
      setSelectedItem(payload);
      emit('trigger-event', { name: 'open-click', event: { folder: payload } });
    };

    const handleEdit = (item) => {
      const payload = item?._original ?? item;
      setSelectedItem(payload);
      emit('trigger-event', { name: 'edit-click', event: { folder: payload } });
    };

    const handleNameClick = (item) => {
      const payload = item?._original ?? item;
      setSelectedItem(payload);
      emit('trigger-event', { name: 'name-click', event: { folder: payload } });
    };

    const handleShareLink = (item) => {
      const payload = item?._original ?? item;
      setSelectedItem(payload);
      emit('trigger-event', { name: 'share-link-click', event: { folder: payload } });
    };

    return {
      processedItems,
      containerStyle,
      cardStyle,
      openButtonStyle,
      editButtonStyle,
      shareLinkButtonStyle,
      folderNameStyle,
      labelStyle,
      valueStyle,
      checkedBoxStyle,
      uncheckedBoxStyle,
      emptyStateStyle,
      getAiPolicyText,
      getAiPolicyIcon,
      handleOpen,
      handleEdit,
      handleNameClick,
      handleShareLink,
      setShareHover,
      selectedItem,
      itemCount,
      /* wwEditor:start */
      isEditing,
      /* wwEditor:end */
    };
  },
};
</script>

<style scoped>
.folder-card-list {
  width: 100%;
  box-sizing: border-box;
}

.folder-card {
  width: 100%;
  box-sizing: border-box;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  background: var(--fcl-card-bg, #ffffff);
  border: 1px solid var(--fcl-card-border, #e5e7eb);
  border-radius: var(--fcl-card-radius, 8px);
  font-size: var(--fcl-font-size, 14px);
}

.card-actions {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 8px;
}

.btn-action {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 6px 18px;
  border-radius: 999px;
  font-size: var(--fcl-font-size, 14px);
  font-weight: 500;
  line-height: 1.4;
  white-space: nowrap;
  cursor: pointer;
  transition: opacity 0.15s ease;
  user-select: none;
}

.btn-action:hover { opacity: 0.82; }
.btn-action:active { opacity: 0.65; }

.btn-open { border: 1.5px solid transparent; }
.btn-edit { border: 1.5px solid; }

.folder-name-field { margin-top: 2px; }

.folder-name {
  color: var(--fcl-name-color, #2d6a4f);
  font-size: var(--fcl-font-size, 14px);
  font-weight: 600;
  text-decoration: underline;
  text-underline-offset: 2px;
  cursor: pointer;
  transition: opacity 0.15s ease;
  display: inline;
}

.folder-name:hover { opacity: 0.72; }

.card-field {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  min-height: 22px;
}

.field-label {
  color: var(--fcl-label-color, #6b7280);
  font-size: var(--fcl-font-size, 14px);
  font-weight: 400;
  flex-shrink: 0;
}

.field-value {
  color: var(--fcl-value-color, #111827);
  font-size: var(--fcl-font-size, 14px);
  font-weight: 500;
  text-align: right;
  display: flex;
  align-items: center;
  gap: 4px;
}

.ai-policy-value {
  display: flex;
  align-items: center;
  gap: 4px;
}

.ai-policy-icon {
  font-size: 1em;
  line-height: 1;
  flex-shrink: 0;
}

.checkbox-value {
  display: flex;
  align-items: center;
  justify-content: flex-end;
}

.checkbox-box {
  width: 16px;
  height: 16px;
  border-radius: 3px;
  border: 1.5px solid #d1d5db;
  background: #ffffff;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  pointer-events: none;
  user-select: none;
}

.btn-share-link {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border-radius: 6px;
  border: 1.5px solid;
  background: transparent;
  cursor: pointer;
  transition: opacity 0.15s ease, background-color 0.15s ease;
  flex-shrink: 0;
  padding: 0;
}

.btn-share-link:hover {
  opacity: 0.75;
}

.btn-share-link:active {
  opacity: 0.5;
}

.empty-state {
  width: 100%;
  padding: 32px 16px;
  text-align: center;
  color: var(--fcl-label-color, #6b7280);
  font-size: var(--fcl-font-size, 14px);
  box-sizing: border-box;
}
</style>
