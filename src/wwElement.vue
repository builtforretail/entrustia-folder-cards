<template>
  <div class="pp-card-list" :style="containerStyle">

    <!-- Filter Bar -->
    <div class="filter-bar">
      <div class="filter-search-wrap">
        <svg class="filter-search-icon" width="14" height="14" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <circle cx="11" cy="11" r="8" stroke="#9ca3af" stroke-width="2"/>
          <path d="M21 21l-4.35-4.35" stroke="#9ca3af" stroke-width="2" stroke-linecap="round"/>
        </svg>
        <input
          class="filter-search"
          type="text"
          placeholder="Page Name"
          :value="searchText"
          @input="onSearchInput"
        />
        <button v-if="searchText" class="filter-clear-btn" type="button" @click="clearSearch" aria-label="Clear search">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M18 6L6 18M6 6l12 12" stroke="#9ca3af" stroke-width="2" stroke-linecap="round"/>
          </svg>
        </button>
      </div>
      <div class="filter-select-wrap">
        <select class="filter-select" :value="statusFilter" @change="onStatusChange">
          <option value="all">All</option>
          <option value="enabled">Active Public Pages</option>
          <option value="disabled">Inactive Public Pages</option>
        </select>
        <button v-if="statusFilter !== 'all'" class="filter-clear-btn filter-clear-select" type="button" @click="clearStatus" aria-label="Clear status">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M18 6L6 18M6 6l12 12" stroke="#9ca3af" stroke-width="2" stroke-linecap="round"/>
          </svg>
        </button>
      </div>
      <button class="filter-reset" type="button" @click="resetFilters">Reset</button>
    </div>

    <!-- Cards -->
    <div
      v-for="item in filteredItems"
      :key="item.id"
      class="pp-card"
      :style="cardStyle"
    >
      <!-- Action Row -->
      <div class="card-actions">
        <button
          class="btn-primary"
          :style="getEditButtonStyle(item.id)"
          type="button"
          @click="handleEdit(item)"
          @mouseenter="setHover(item.id, 'edit', true)"
          @mouseleave="setHover(item.id, 'edit', false)"
          @mousedown="setActive(item.id, 'edit', true)"
          @mouseup="setActive(item.id, 'edit', false)"
        >
          Edit
        </button>
      </div>

      <!-- Active -->
      <div class="card-field readonly-field">
        <span class="field-label" :style="labelStyle">Active</span>
        <span class="field-value">
          <span class="status-badge" :style="item.is_enabled ? activeBadgeStyle : inactiveBadgeStyle">
            {{ item.is_enabled ? 'Active' : 'Inactive' }}
          </span>
        </span>
      </div>

      <!-- Page Name + Folder Name -->
      <div class="card-field readonly-field">
        <span class="field-label" :style="labelStyle">Page Name</span>
        <span class="field-value field-value-right">
          <span class="page-name-text" :style="valueStyle">{{ item.name }}</span>
          <span class="folder-name-sub" :style="labelStyle">Folder: {{ item.folder_name }}</span>
        </span>
      </div>

      <!-- File Types -->
      <div class="card-field readonly-field">
        <span class="field-label" :style="labelStyle">File Types</span>
        <span class="field-value file-types-value">
          <span v-for="badge in item.fileBadges" :key="badge" class="file-badge" :style="fileBadgeStyle">{{ badge }}</span>
          <span v-if="!item.fileBadges.length" :style="valueStyle">—</span>
        </span>
      </div>

      <!-- Max Size -->
      <div class="card-field readonly-field">
        <span class="field-label" :style="labelStyle">Max Size</span>
        <span class="field-value" :style="valueStyle">{{ item.maxSizeDisplay }}</span>
      </div>

      <!-- Page Instructions -->
      <div class="card-field">
        <span class="field-label" :style="labelStyle">Page Instructions</span>
        <span class="field-value">
          <button
            class="btn-outline-sm"
            :style="getInstructionsButtonStyle(item.id)"
            type="button"
            @click="handleInstructions(item)"
            @mouseenter="setHover(item.id, 'instructions', true)"
            @mouseleave="setHover(item.id, 'instructions', false)"
            @mousedown="setActive(item.id, 'instructions', true)"
            @mouseup="setActive(item.id, 'instructions', false)"
          >
            Click to Edit
          </button>
        </span>
      </div>

      <!-- Expiry Date -->
      <div class="card-field readonly-field">
        <span class="field-label" :style="labelStyle">Expiry Date</span>
        <span class="field-value" :style="valueStyle">{{ item.expiryDisplay }}</span>
      </div>

      <!-- Captcha -->
      <div class="card-field readonly-field">
        <span class="field-label" :style="labelStyle">Captcha?</span>
        <span class="field-value checkbox-value">
          <span class="checkbox-box" :style="item.captcha_required ? checkedBoxStyle : uncheckedBoxStyle" aria-hidden="true">
            <svg v-if="item.captcha_required" width="10" height="8" viewBox="0 0 10 8" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M1 4L3.5 6.5L9 1" stroke="white" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </span>
        </span>
      </div>

      <!-- Share + Link + Embed -->
      <div class="card-actions-bottom">
        <button
          class="btn-icon-action"
          :style="getIconButtonStyle(item.id, 'share')"
          type="button"
          @click="handleShare(item)"
          @mouseenter="setHover(item.id, 'share', true)"
          @mouseleave="setHover(item.id, 'share', false)"
          @mousedown="setActive(item.id, 'share', true)"
          @mouseup="setActive(item.id, 'share', false)"
        >
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M4 12v8a2 2 0 002 2h12a2 2 0 002-2v-8M16 6l-4-4-4 4M12 2v13" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          Share
        </button>
        <button
          class="btn-icon-action"
          :style="getIconButtonStyle(item.id, 'link')"
          type="button"
          @click="handleLink(item)"
          @mouseenter="setHover(item.id, 'link', true)"
          @mouseleave="setHover(item.id, 'link', false)"
          @mousedown="setActive(item.id, 'link', true)"
          @mouseup="setActive(item.id, 'link', false)"
        >
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <rect x="9" y="9" width="13" height="13" rx="2" ry="2" stroke="currentColor" stroke-width="2"/>
            <path d="M5 15H4a2 2 0 01-2-2V4a2 2 0 012-2h9a2 2 0 012 2v1" stroke="currentColor" stroke-width="2"/>
          </svg>
          Link
        </button>
        <button
          class="btn-icon-action"
          :style="getIconButtonStyle(item.id, 'embed')"
          type="button"
          @click="handleEmbed(item)"
          @mouseenter="setHover(item.id, 'embed', true)"
          @mouseleave="setHover(item.id, 'embed', false)"
          @mousedown="setActive(item.id, 'embed', true)"
          @mouseup="setActive(item.id, 'embed', false)"
        >
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <polyline points="16 18 22 12 16 6" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            <polyline points="8 6 2 12 8 18" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          Embed
        </button>
      </div>
    </div>

    <!-- Empty state -->
    <div v-if="!filteredItems.length" class="empty-state" :style="emptyStateStyle">
      No pages match your filters.
    </div>

  </div>
</template>

<script>
import { computed, ref, watch } from 'vue';

export default {
  name: 'PublicPagesCardList',

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
    const isEditing = computed(() => props.wwEditorState && props.wwEditorState.isEditing);
    /* wwEditor:end */

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

    const { value: filteredCount, setValue: setFilteredCount } =
      wwLib.wwVariable.useComponentVariable({
        uid: props.uid,
        name: 'filteredCount',
        type: 'number',
        defaultValue: 0,
      });

    const searchText = ref('');
    const statusFilter = ref('all');

    const onSearchInput = function(e) { searchText.value = e.target.value; };
    const onStatusChange = function(e) { statusFilter.value = e.target.value; };
    const clearSearch = function() { searchText.value = ''; };
    const clearStatus = function() { statusFilter.value = 'all'; };
    const resetFilters = function() { searchText.value = ''; statusFilter.value = 'all'; };

    const hoverState = ref({});
    const activeState = ref({});

    const setHover = function(id, btn, val) {
      hoverState.value = Object.assign({}, hoverState.value, { [id + '-' + btn]: val });
    };

    const setActive = function(id, btn, val) {
      activeState.value = Object.assign({}, activeState.value, { [id + '-' + btn]: val });
    };

    const darken = function(hex, amount) {
      const h = (hex || '#2d6a4f').replace('#', '');
      const full = h.length === 3 ? h[0]+h[0]+h[1]+h[1]+h[2]+h[2] : h;
      const num = parseInt(full, 16);
      const r = Math.max(0, (num >> 16) - amount);
      const g = Math.max(0, ((num >> 8) & 0xff) - amount);
      const b = Math.max(0, (num & 0xff) - amount);
      return '#' + [r, g, b].map(function(v) { return v.toString(16).padStart(2, '0'); }).join('');
    };

    const getMimeBadges = function(patterns) {
      if (!patterns || !Array.isArray(patterns) || !patterns.length) return [];
      const badges = [];
      const seen = {};
      patterns.forEach(function(p) {
        let val = '';
        if (p && typeof p === 'object') {
          val = p.value || '';
        } else if (typeof p === 'string') {
          val = p;
        }
        if (val === 'application/pdf' && !seen['PDF']) { badges.push('PDF'); seen['PDF'] = true; }
        else if (val === 'image/*' && !seen['Images']) { badges.push('Images'); seen['Images'] = true; }
        else if (val === 'application/*' && !seen['Office']) { badges.push('Office'); seen['Office'] = true; }
        else if (val === 'text/*' && !seen['Text']) { badges.push('Text'); seen['Text'] = true; }
      });
      return badges;
    };

    const formatExpiry = function(val) {
      if (!val) return 'mm/dd/yyyy';
      try {
        const parts = String(val).split('T')[0].split('-');
        if (parts.length !== 3) return 'mm/dd/yyyy';
        return parts[1] + '/' + parts[2] + '/' + parts[0];
      } catch(e) { return 'mm/dd/yyyy'; }
    };

    const formatMaxBytes = function(bytes) {
      if (!bytes && bytes !== 0) return '—';
      return Math.round(bytes / (1024 * 1024)) + ' MB';
    };

    const processedItems = computed(function() {
      const items = Array.isArray(props.content && props.content.data) ? props.content.data : [];
      const formulaObj = wwLib.wwFormula.useFormula();
      const resolveMappingFormula = formulaObj.resolveMappingFormula;

      return items.map(function(item) {
        const id = resolveMappingFormula(props.content && props.content.dataIdFormula, item) || item && item.id;
        const name = resolveMappingFormula(props.content && props.content.dataNameFormula, item) || item && item.name;
        const folder_name = resolveMappingFormula(props.content && props.content.dataFolderNameFormula, item) || item && item.folder_name;
        const is_enabled = resolveMappingFormula(props.content && props.content.dataIsEnabledFormula, item);
        const is_enabled_val = is_enabled !== null && is_enabled !== undefined ? is_enabled : (item && item.is_enabled);
        const allowed_mime_patterns = resolveMappingFormula(props.content && props.content.dataAllowedMimeFormula, item) || item && item.allowed_mime_patterns || [];
        const max_bytes = resolveMappingFormula(props.content && props.content.dataMaxBytesFormula, item) || item && item.max_bytes;
        const instructions = resolveMappingFormula(props.content && props.content.dataInstructionsFormula, item) || item && item.instructions || '';
        const expires_at = resolveMappingFormula(props.content && props.content.dataExpiresAtFormula, item) || item && item.expires_at;
        const captcha_required = resolveMappingFormula(props.content && props.content.dataCaptchaRequiredFormula, item);
        const captcha_val = captcha_required !== null && captcha_required !== undefined ? captcha_required : (item && item.captcha_required);
        const token = resolveMappingFormula(props.content && props.content.dataTokenFormula, item) || item && item.token || '';

        return {
          id: id || ('item-' + Math.random()),
          name: name || 'Untitled',
          folder_name: folder_name || '',
          is_enabled: Boolean(is_enabled_val),
          allowed_mime_patterns: allowed_mime_patterns,
          max_bytes: max_bytes || 0,
          instructions: instructions,
          expires_at: expires_at || null,
          captcha_required: Boolean(captcha_val),
          token: token,
          fileBadges: getMimeBadges(allowed_mime_patterns),
          maxSizeDisplay: formatMaxBytes(max_bytes || item && item.max_bytes),
          expiryDisplay: formatExpiry(expires_at || item && item.expires_at),
          _original: item,
        };
      });
    });

    const filteredItems = computed(function() {
      const items = Array.isArray(processedItems.value) ? processedItems.value : [];
      const search = (searchText.value || '').toLowerCase();
      const status = statusFilter.value;

      return items.filter(function(item) {
        const nameMatch = !search || (item.name || '').toLowerCase().indexOf(search) !== -1;
        const statusMatch = status === 'all' ||
          (status === 'enabled' && item.is_enabled) ||
          (status === 'disabled' && !item.is_enabled);
        return nameMatch && statusMatch;
      });
    });

    watch(processedItems, function(items) { setItemCount(items.length || 0); }, { immediate: true });
    watch(filteredItems, function(items) { setFilteredCount(items.length || 0); }, { immediate: true });

    const resolvedPrimaryColor = computed(function() { return (props.content && props.content.primaryColor) || '#2d6a4f'; });
    const resolvedOutlineColor = computed(function() { return (props.content && props.content.outlineColor) || '#2d6a4f'; });

    const containerStyle = computed(function() {
      return {
        '--pp-primary': resolvedPrimaryColor.value,
        '--pp-outline': resolvedOutlineColor.value,
        '--pp-card-bg': (props.content && props.content.cardBackground) || '#ffffff',
        '--pp-card-border': (props.content && props.content.cardBorderColor) || '#e5e7eb',
        '--pp-card-radius': ((props.content && props.content.cardBorderRadius) || 8) + 'px',
        '--pp-label-color': (props.content && props.content.labelTextColor) || '#6b7280',
        '--pp-value-color': (props.content && props.content.valueTextColor) || '#111827',
        '--pp-gap': ((props.content && props.content.cardGap) || 32) + 'px',
        '--pp-font-size': ((props.content && props.content.fontSize) || 14) + 'px',
        display: 'flex',
        flexDirection: 'column',
        gap: ((props.content && props.content.cardGap) || 32) + 'px',
        width: '100%',
        padding: '4px 3px 8px 3px',
        boxSizing: 'border-box',
      };
    });

    const cardStyle = computed(function() {
      return {
        background: (props.content && props.content.cardBackground) || '#ffffff',
        border: '1px solid ' + ((props.content && props.content.cardBorderColor) || '#e5e7eb'),
        borderRadius: ((props.content && props.content.cardBorderRadius) || 8) + 'px',
        fontSize: ((props.content && props.content.fontSize) || 14) + 'px',
      };
    });

    const labelStyle = computed(function() {
      return {
        color: (props.content && props.content.labelTextColor) || '#6b7280',
        fontSize: ((props.content && props.content.fontSize) || 14) + 'px',
      };
    });

    const valueStyle = computed(function() {
      return {
        color: (props.content && props.content.valueTextColor) || '#111827',
        fontSize: ((props.content && props.content.fontSize) || 14) + 'px',
      };
    });

    const activeBadgeStyle = computed(function() {
      return { backgroundColor: '#d1fae5', color: '#065f46', fontSize: ((props.content && props.content.fontSize) || 14) + 'px' };
    });

    const inactiveBadgeStyle = computed(function() {
      return { backgroundColor: '#f3f4f6', color: '#6b7280', fontSize: ((props.content && props.content.fontSize) || 14) + 'px' };
    });

    const fileBadgeStyle = computed(function() {
      return {
        backgroundColor: resolvedPrimaryColor.value,
        color: '#ffffff',
        fontSize: (((props.content && props.content.fontSize) || 14) - 1) + 'px',
      };
    });

    const checkedBoxStyle = computed(function() {
      return { backgroundColor: resolvedPrimaryColor.value, borderColor: resolvedPrimaryColor.value };
    });

    const uncheckedBoxStyle = computed(function() {
      return { backgroundColor: '#ffffff', borderColor: '#d1d5db' };
    });

    const emptyStateStyle = computed(function() {
      return {
        color: (props.content && props.content.labelTextColor) || '#6b7280',
        fontSize: ((props.content && props.content.fontSize) || 14) + 'px',
      };
    });

    const getEditButtonStyle = function(id) {
      const isActive = activeState.value[id + '-edit'];
      const isHovered = hoverState.value[id + '-edit'];
      const base = resolvedPrimaryColor.value;
      const bg = isActive ? darken(base, 40) : (isHovered ? darken(base, 20) : base);
      return {
        backgroundColor: bg,
        color: '#ffffff',
        borderColor: bg,
        fontSize: ((props.content && props.content.fontSize) || 14) + 'px',
        boxShadow: (isHovered && !isActive) ? '0 2px 8px rgba(0,0,0,0.18)' : '0 1px 3px rgba(0,0,0,0.10)',
        transform: isActive ? 'scale(0.97)' : 'scale(1)',
        transition: 'background-color 0.15s ease, box-shadow 0.15s ease, transform 0.1s ease',
      };
    };

    const getInstructionsButtonStyle = function(id) {
      const isActive = activeState.value[id + '-instructions'];
      const isHovered = hoverState.value[id + '-instructions'];
      const base = resolvedOutlineColor.value;
      const darkened = isActive ? darken(base, 40) : (isHovered ? darken(base, 20) : base);
      return {
        backgroundColor: isHovered ? darkened : '#ffffff',
        color: isHovered ? '#ffffff' : base,
        borderColor: darkened,
        fontSize: (((props.content && props.content.fontSize) || 14) - 1) + 'px',
        boxShadow: (isHovered && !isActive) ? '0 2px 8px rgba(0,0,0,0.15)' : 'none',
        transform: isActive ? 'scale(0.97)' : 'scale(1)',
        transition: 'background-color 0.15s ease, color 0.15s ease, box-shadow 0.15s ease, transform 0.1s ease',
      };
    };

    const getIconButtonStyle = function(id, key) {
      const isActive = activeState.value[id + '-' + key];
      const isHovered = hoverState.value[id + '-' + key];
      const base = resolvedOutlineColor.value;
      return {
        color: isHovered ? darken(base, 20) : base,
        backgroundColor: isHovered ? (isActive ? '#e5e7eb' : '#f3f4f6') : 'transparent',
        transform: isActive ? 'scale(0.95)' : 'scale(1)',
        transition: 'color 0.15s ease, background-color 0.15s ease, transform 0.1s ease',
      };
    };

    const handleEdit = function(item) {
      const payload = item._original || item;
      setSelectedItem(payload);
      emit('trigger-event', { name: 'edit-click', event: { page: payload } });
    };

    const handleInstructions = function(item) {
      const payload = item._original || item;
      setSelectedItem(payload);
      emit('trigger-event', { name: 'instructions-click', event: { page: payload } });
    };

    const handleShare = function(item) {
      const payload = item._original || item;
      setSelectedItem(payload);
      emit('trigger-event', { name: 'share-click', event: { page: payload } });
    };

    const handleLink = function(item) {
      const payload = item._original || item;
      setSelectedItem(payload);
      emit('trigger-event', { name: 'link-click', event: { page: payload } });
    };

    const handleEmbed = function(item) {
      const payload = item._original || item;
      setSelectedItem(payload);
      emit('trigger-event', { name: 'embed-click', event: { page: payload } });
    };

    return {
      props,
      processedItems,
      filteredItems,
      searchText,
      statusFilter,
      onSearchInput,
      onStatusChange,
      clearSearch,
      clearStatus,
      resetFilters,
      containerStyle,
      cardStyle,
      labelStyle,
      valueStyle,
      activeBadgeStyle,
      inactiveBadgeStyle,
      fileBadgeStyle,
      checkedBoxStyle,
      uncheckedBoxStyle,
      emptyStateStyle,
      getEditButtonStyle,
      getInstructionsButtonStyle,
      getIconButtonStyle,
      handleEdit,
      handleInstructions,
      handleShare,
      handleLink,
      handleEmbed,
      setHover,
      setActive,
      selectedItem,
      itemCount,
      filteredCount,
      /* wwEditor:start */
      isEditing,
      /* wwEditor:end */
    };
  },
};
</script>

<style scoped>
.pp-card-list {
  width: 100%;
  box-sizing: border-box;
}

.filter-bar {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 8px;
  margin-bottom: 10px;
  width: 100%;
}

.filter-search-wrap {
  position: relative;
  flex: 1;
  min-width: 0;
  display: flex;
  align-items: center;
}

.filter-search-icon {
  position: absolute;
  left: 10px;
  pointer-events: none;
  flex-shrink: 0;
}

.filter-search {
  width: 100%;
  padding: 7px 28px 7px 30px;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
  font-size: 13px;
  color: #111827;
  background: #ffffff;
  outline: none;
  box-sizing: border-box;
}

.filter-search:focus {
  border-color: #2d6a4f;
}

.filter-clear-btn {
  position: absolute;
  right: 8px;
  background: none;
  border: none;
  padding: 2px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
}

.filter-clear-btn:hover {
  background: #f3f4f6;
}

.filter-select-wrap {
  position: relative;
  flex: 1;
  min-width: 0;
  display: flex;
  align-items: center;
}

.filter-select {
  width: 100%;
  padding: 7px 28px 7px 10px;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
  font-size: 13px;
  color: #111827;
  background: #ffffff;
  outline: none;
  appearance: none;
  box-sizing: border-box;
  cursor: pointer;
}

.filter-select:focus {
  border-color: #2d6a4f;
}

.filter-clear-select {
  right: 20px;
}

.filter-reset {
  background: none;
  border: none;
  color: #2d6a4f;
  font-size: 13px;
  font-weight: 500;
  text-decoration: underline;
  cursor: pointer;
  white-space: nowrap;
  padding: 0 2px;
  flex-shrink: 0;
}

.filter-reset:hover {
  color: #1a4a35;
}

.pp-card {
  width: 100%;
  box-sizing: border-box;
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 0;
  background: #ffffff;
  border: 1.5px solid #9ca3af;
  border-radius: var(--pp-card-radius, 8px);
  font-size: var(--pp-font-size, 14px);
  overflow: hidden;
  box-shadow: 0 3px 10px rgba(0,0,0,0.10);
  margin-bottom: 16px;
}

.card-actions {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 10px;
  padding: 12px 16px 10px 16px;
  background: #ffffff;
  border-bottom: 1px solid #e5e7eb;
}

.btn-primary {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 6px 20px;
  border-radius: 999px;
  font-size: var(--pp-font-size, 14px);
  font-weight: 600;
  line-height: 1.4;
  white-space: nowrap;
  cursor: pointer;
  user-select: none;
  border: 2px solid;
}

.btn-outline-sm {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 4px 12px;
  border-radius: 999px;
  font-weight: 500;
  line-height: 1.4;
  white-space: nowrap;
  cursor: pointer;
  user-select: none;
  border: 1.5px solid;
}

.readonly-field {
  background: #ffffff;
}

.card-field {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  min-height: 36px;
  padding: 6px 16px;
  border-bottom: 1px solid var(--pp-card-border, #e5e7eb);
}

.field-label {
  color: var(--pp-label-color, #6b7280);
  font-size: calc(var(--pp-font-size, 14px) - 1px);
  font-weight: 600;
  flex-shrink: 0;
  text-transform: none;
  letter-spacing: 0;
}

.field-value {
  color: var(--pp-value-color, #111827);
  font-size: var(--pp-font-size, 14px);
  font-weight: 400;
  text-align: right;
  display: flex;
  align-items: center;
  gap: 4px;
  flex-wrap: wrap;
  justify-content: flex-end;
}

.field-value-right {
  flex-direction: column;
  align-items: flex-end;
  gap: 1px;
}

.page-name-text {
  font-weight: 500;
  text-align: right;
}

.folder-name-sub {
  font-size: calc(var(--pp-font-size, 14px) - 2px);
  font-style: italic;
  color: var(--pp-label-color, #6b7280);
  text-align: right;
}

.status-badge {
  display: inline-flex;
  align-items: center;
  padding: 2px 10px;
  border-radius: 999px;
  font-size: var(--pp-font-size, 14px);
  font-weight: 500;
}

.file-types-value {
  flex-wrap: wrap;
  justify-content: flex-end;
  gap: 4px;
  max-width: 60%;
}

.file-badge {
  display: inline-flex;
  align-items: center;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: calc(var(--pp-font-size, 14px) - 1px);
  font-weight: 600;
  color: #ffffff;
  background: var(--pp-primary, #2d6a4f);
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

.card-actions-bottom {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 8px;
  padding: 8px 16px 10px 16px;
  background: #ffffff;
  border-top: 1px solid #e5e7eb;
}

.btn-icon-action {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 5px 12px;
  border-radius: 6px;
  font-size: var(--pp-font-size, 14px);
  font-weight: 500;
  cursor: pointer;
  user-select: none;
  border: none;
  background: transparent;
}

.empty-state {
  width: 100%;
  padding: 32px 16px;
  text-align: center;
  color: var(--pp-label-color, #6b7280);
  font-size: var(--pp-font-size, 14px);
  box-sizing: border-box;
}
</style>
