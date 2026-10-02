# Bug Fix Report: Mobile Song Selector Click Issue

## File Yang Diubah
- `index.html` (CSS dan struktur stacking context)

## ROOT CAUSE Bug

**Problem Statement:**
Tombol "Song 1: Daylight" dan "Song 2: you!" tidak bisa diklik pada mobile viewport (≤640px), meskipun tombol terlihat normal secara visual.

**Root Cause Analysis:**
1. **Stacking Context Issue**: `.nav-wrapper` memiliki `position: fixed` dengan `z-index: 900`, menciptakan stacking context baru yang mencakup seluruh navbar dan mobile menu.

2. **Mobile Menu Positioning**: `.mobile-menu-overlay` pada mobile tidak memiliki `position: absolute` dengan positioning yang jelas, sehingga dapat secara tidak sengaja menutupi area di bawahnya melalui stacking context yang salah.

3. **Pointer Events**: Meskipun `.track-tab` sudah memiliki `pointer-events: auto`, parent container `.nav-wrapper` dengan `position: fixed` dapat menciptakan barrier untuk pointer events dari elemen di luar struktur nav.

4. **Z-index Hierarchy**: Tidak ada kontrol yang jelas untuk memastikan song selector (z-index 20-21) berada di atas fixed navbar (z-index 900) pada mobile.

## Perubahan Yang Dilakukan

### 1. Mobile Menu Overlay - Positioning Fix (Line 161-183)
```css
.mobile-menu-overlay {
    position: absolute;      /* NEW: Mengubah dari default ke absolute */
    top: 100%;              /* NEW: Positioning relative to nav-wrapper */
    left: 0;                /* NEW: Align ke kiri */
    right: 0;               /* NEW: Stretch penuh lebar */
    width: auto;            /* NEW: Biarkan natural width */
    z-index: 800;           /* NEW: Z-index lebih rendah dari buttons */
}
```

Alasan: Dengan `position: absolute`, mobile menu akan di-positioning relative ke `.nav-wrapper` (yang memiliki `position: fixed`), bukan mempengaruhi flow document. Ini mencegah overlay menghadang elemen lain di bawahnya.

### 2. Nav Wrapper Pointer Events - Mobile Breakpoint (Line 1422-1427)
```css
@media (max-width: 640px) {
    .nav-wrapper {
        pointer-events: none;  /* NEW: Nav wrapper tidak menerima pointer events */
    }
    .nav-body {
        pointer-events: auto;  /* NEW: Hanya nav-body yang menerima events */
    }
}
```

Alasan: Dengan `pointer-events: none` pada `.nav-wrapper`, elemen di bawahnya (song selector) dapat menerima click/tap events tanpa terhadang. Hanya `.nav-body` (navbar items) yang perlu menerima events.

### 3. Track Tab Pointer Events (Line 1116)
```css
.track-tab {
    pointer-events: auto !important;  /* Changed from auto to auto !important */
}
```

Alasan: `!important` memastikan property ini tidak ditimpa oleh CSS rules lain pada mobile breakpoint.

### 4. Spotify Player Z-index (Line 1093)
```css
.spotify-player {
    position: relative;
    width: 100%;
    z-index: 1;  /* NEW: Explicit z-index untuk stacking context clarity */
}
```

Alasan: Memberikan z-index explicit untuk memastikan stacking context jelas dan terukur.

### 5. Mobile Menu Overlay Active State (Line 1436)
```css
@media (max-width: 640px) {
    .mobile-menu-overlay {
        display: flex;
        pointer-events: auto;  /* NEW: Explicitly set untuk mobile */
    }
}
```

Alasan: Memastikan mobile menu hanya menerima events ketika di-display pada mobile.

## Perubahan yang TIDAK dilakukan

1. **Tidak mengubah logic JavaScript**: `switchTrack()` function tetap sama
2. **Tidak menghapus functionality**: Semua fitur existing tetap berjalan
3. **Tidak merubah visual design**: Hanya CSS positioning dan pointer-events
4. **Tidak menghapus responsive breakpoints**: Semua breakpoint (640px, 900px) tetap berjalan
5. **Tidak mengubah HTML structure**: DOM structure tetap sama

## Summary

Bug ini terjadi karena **stacking context hierarchy yang tidak jelas** antara fixed navbar (z-index 900) dan song selector buttons (z-index 20-21) pada mobile. 

Solusi dilakukan dengan:
1. Positioning `.mobile-menu-overlay` secara absolute agar tidak menciptakan barrier
2. Menggunakan `pointer-events: none` pada `.nav-wrapper` untuk mengizinkan pointer events pass-through ke elemen di bawahnya pada mobile
3. Explicit z-index pada `.spotify-player` untuk clarity

Perubahan ini **minimal, targeted, dan tidak merusak existing functionality** pada tablet dan desktop.
