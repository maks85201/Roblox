<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<meta name="theme-color" content="#00B87C">
<meta name="apple-mobile-web-app-capable" content="yes">
<title>Попутчик — голосовой помощник</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<style>
  @import url('https://cdn.jsdelivr.net/npm/fontsource-inter@5.0.0/index.css');
  * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
  :root {
    --base-font-size: 16px;
    --green: #00B87C; --green-dark: #009966; --green-light: #E6F7F0;
    --bg: #F5F7FA; --white: #FFFFFF; --text: #1A2B49;
    --text-secondary: #6B7C93; --border: #E1E5EB;
    --shadow: 0 2px 12px rgba(0,0,0,0.08); --shadow-lg: 0 8px 32px rgba(0,0,0,0.12);
  }
  html { font-size: var(--base-font-size); }
  body { font-family: 'Inter', -apple-system, sans-serif; background: var(--bg); color: var(--text); min-height: 100vh; padding-bottom: 80px; font-size: 1rem; line-height: 1.5; }
  body.large-text { --base-font-size: 20px; }
  body.high-contrast { --bg: #000; --white: #000; --text: #FFF; --text-secondary: #E0E0E0; --border: #FFF; --green: #00FF99; --green-dark: #00CC77; --green-light: #003322; }
  body.high-contrast .trip-card, body.high-contrast .accordion-item, body.high-contrast .search-form, body.high-contrast .profile-menu-item, body.high-contrast .map-modal-content, body.high-contrast .route-header, body.high-contrast .alert-banner, body.high-contrast .profile-stats, body.high-contrast .bottom-nav, body.high-contrast .header, body.high-contrast .bottom-actions, body.high-contrast .request-card, body.high-contrast .ai-modal-content, body.high-contrast .access-modal-content { border: 2px solid #FFF; }

  .screen { display: none; min-height: 100vh; animation: fadeIn 0.25s ease; padding-bottom: 90px; }
  .screen.active { display: block; }
  @keyframes fadeIn { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }
  @keyframes slideUp { from { opacity: 0; transform: translateY(100%); } to { opacity: 1; transform: translateY(0); } }
  @keyframes pulse { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.05); } }
  @keyframes typing { 0%, 60%, 100% { transform: translateY(0); opacity: 0.4; } 30% { transform: translateY(-8px); opacity: 1; } }
  @keyframes wave { 0%, 100% { height: 8px; } 50% { height: 28px; } }
  @keyframes ripple { 0% { transform: scale(1); opacity: 0.6; } 100% { transform: scale(2.5); opacity: 0; } }
  @keyframes subtitleFade { 0% { opacity: 0; transform: translateY(10px); } 100% { opacity: 1; transform: translateY(0); } }

  .accessibility-btn { position: fixed; top: 12px; right: 12px; width: 48px; height: 48px; border-radius: 50%; background: var(--green); color: white; border: none; cursor: pointer; z-index: 150; display: flex; align-items: center; justify-content: center; box-shadow: var(--shadow-lg); }
  .accessibility-btn:active { transform: scale(0.92); }

  .header { background: var(--white); padding: 12px 16px; display: flex; align-items: center; gap: 12px; position: sticky; top: 0; z-index: 100; border-bottom: 1px solid var(--border); }
  .header-back { width: 40px; height: 40px; display: flex; align-items: center; justify-content: center; cursor: pointer; border-radius: 50%; }
  .header-back:active { background: var(--bg); }
  .header-title { font-size: 1.125rem; font-weight: 700; color: var(--text); }

  .main-header { background: var(--white); padding: 20px 16px 16px; }
  .greeting { display: flex; align-items: center; gap: 10px; margin-bottom: 16px; }
  .greeting-icon { width: 40px; height: 40px; }
  .greeting h1 { font-size: 1.375rem; font-weight: 800; color: var(--text); }
  .main-title { font-size: 1.25rem; font-weight: 800; line-height: 1.3; color: var(--text); margin-bottom: 20px; }

  .type-toggle { display: flex; background: var(--bg); border-radius: 12px; padding: 4px; margin-bottom: 16px; }
  .type-toggle-btn { flex: 1; padding: 12px; text-align: center; border-radius: 10px; cursor: pointer; font-size: 0.875rem; font-weight: 600; color: var(--text-secondary); transition: all 0.2s; }
  .type-toggle-btn.active { background: var(--white); color: var(--green); box-shadow: 0 2px 8px rgba(0,0,0,0.08); }

  .search-form { background: var(--white); border-radius: 20px; padding: 20px; margin: 0 16px 20px; box-shadow: var(--shadow); border: 2px solid var(--green); }
  .form-group { padding: 14px 0; border-bottom: 1px solid var(--border); cursor: pointer; position: relative; }
  .form-group:last-child { border-bottom: none; }
  .form-group:active { opacity: 0.7; }
  .form-label { font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 4px; font-weight: 500; }
  .form-value { font-size: 1rem; font-weight: 600; color: var(--text); }
  .form-value.placeholder { color: var(--text-secondary); font-weight: 400; }
  .form-input-field { width: 100%; border: none; outline: none; font-size: 1rem; font-family: inherit; font-weight: 600; color: var(--text); background: transparent; padding: 0; }
  .form-input-field::placeholder { color: var(--text-secondary); font-weight: 400; }
  .search-btn { width: 100%; padding: 18px; background: var(--green); color: white; border: none; border-radius: 16px; font-size: 1.125rem; font-weight: 700; cursor: pointer; margin-top: 16px; font-family: inherit; }
  .search-btn:active { transform: scale(0.98); background: var(--green-dark); }

  .autocomplete-container { position: relative; }
  .autocomplete-list { position: absolute; top: 100%; left: 0; right: 0; background: var(--white); border: 1px solid var(--border); border-radius: 12px; max-height: 200px; overflow-y: auto; z-index: 50; box-shadow: var(--shadow-lg); display: none; }
  .autocomplete-list.show { display: block; }
  .autocomplete-item { padding: 12px 16px; cursor: pointer; border-bottom: 1px solid var(--border); font-size: 0.875rem; color: var(--text); }
  .autocomplete-item:last-child { border-bottom: none; }
  .autocomplete-item:active { background: var(--green-light); }
  .autocomplete-item strong { color: var(--green); }

  .directory-section { padding: 0 16px; margin-bottom: 20px; }
  .directory-header { display: flex; align-items: center; gap: 10px; margin-bottom: 14px; }
  .directory-title { font-size: 1.25rem; font-weight: 800; color: var(--text); }
  .directory-subtitle { font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 14px; line-height: 1.4; }
  .directory-search { width: 100%; padding: 14px 16px; background: var(--white); border: 1.5px solid var(--border); border-radius: 14px; font-size: 1rem; font-family: inherit; color: var(--text); margin-bottom: 12px; outline: none; }
  .directory-search:focus { border-color: var(--green); }
  .directory-search::placeholder { color: var(--text-secondary); }
  .hotline-banner { background: linear-gradient(135deg, #00B87C 0%, #00D68F 100%); border-radius: 16px; padding: 16px 18px; margin-bottom: 14px; color: white; display: flex; align-items: center; gap: 12px; cursor: pointer; text-decoration: none; }
  .hotline-banner:active { transform: scale(0.98); }
  .hotline-icon { width: 48px; height: 48px; background: rgba(255,255,255,0.2); border-radius: 12px; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
  .hotline-content { flex: 1; }
  .hotline-title { font-size: 0.9375rem; font-weight: 700; margin-bottom: 2px; }
  .hotline-number { font-size: 1.125rem; font-weight: 800; }

  .accordion { display: flex; flex-direction: column; gap: 8px; }
  .accordion-item { background: var(--white); border-radius: 16px; overflow: hidden; border: 1px solid var(--border); }
  .accordion-item.open { border-color: var(--green); }
  .accordion-header { padding: 16px 18px; display: flex; align-items: center; gap: 14px; cursor: pointer; }
  .accordion-header:active { background: var(--green-light); }
  .accordion-icon { width: 44px; height: 44px; border-radius: 12px; background: var(--green-light); display: flex; align-items: center; justify-content: center; flex-shrink: 0; color: var(--green); }
  .accordion-item.open .accordion-icon { background: var(--green); color: white; }
  .accordion-info { flex: 1; min-width: 0; }
  .accordion-city { font-size: 1rem; font-weight: 700; color: var(--text); margin-bottom: 2px; }
  .accordion-region { font-size: 0.75rem; color: var(--text-secondary); }
  .accordion-arrow { width: 24px; height: 24px; display: flex; align-items: center; justify-content: center; color: var(--text-secondary); transition: transform 0.3s; flex-shrink: 0; }
  .accordion-item.open .accordion-arrow { transform: rotate(180deg); color: var(--green); }
  .accordion-content { max-height: 0; overflow: hidden; transition: max-height 0.35s ease; }
  .accordion-item.open .accordion-content { max-height: 500px; }
  .accordion-body { padding: 0 18px 18px; border-top: 1px solid var(--border); padding-top: 14px; }
  .info-row { display: flex; align-items: flex-start; gap: 12px; padding: 10px 0; }
  .info-row-icon { width: 36px; height: 36px; border-radius: 10px; background: var(--green-light); display: flex; align-items: center; justify-content: center; flex-shrink: 0; color: var(--green); }
  .info-row-content { flex: 1; min-width: 0; }
  .info-row-label { font-size: 0.75rem; color: var(--text-secondary); margin-bottom: 2px; font-weight: 500; }
  .info-row-value { font-size: 0.9375rem; font-weight: 600; color: var(--text); line-height: 1.4; }
  .call-btn { display: flex; align-items: center; justify-content: center; gap: 8px; width: 100%; padding: 14px; background: var(--green); color: white; border: none; border-radius: 12px; font-size: 1rem; font-weight: 700; cursor: pointer; margin-top: 10px; font-family: inherit; text-decoration: none; }
  .call-btn:active { background: var(--green-dark); transform: scale(0.98); }
  .call-btn.secondary { background: var(--green-light); color: var(--green); }

  .tabs { display: flex; background: var(--white); padding: 0 16px; border-bottom: 1px solid var(--border); overflow-x: auto; }
  .tab { flex: 1; min-width: 100px; padding: 14px 8px; text-align: center; cursor: pointer; position: relative; }
  .tab-label { font-size: 0.875rem; font-weight: 600; color: var(--text-secondary); margin-bottom: 4px; }
  .tab-count { font-size: 0.75rem; color: var(--text-secondary); }
  .tab.active .tab-label { color: var(--green); }
  .tab.active::after { content: ''; position: absolute; bottom: 0; left: 16px; right: 16px; height: 3px; background: var(--green); border-radius: 3px 3px 0 0; }

  .trip-card { background: var(--white); border-radius: 20px; margin: 0 16px 12px; overflow: hidden; box-shadow: var(--shadow); cursor: pointer; }
  .trip-card:active { transform: scale(0.98); }
  .trip-card.disabled { opacity: 0.5; }
  .trip-main { padding: 20px; display: flex; align-items: center; gap: 16px; }
  .trip-times { display: flex; flex-direction: column; align-items: flex-end; min-width: 70px; }
  .trip-departure, .trip-arrival { font-size: 1.25rem; font-weight: 800; color: var(--text); }
  .trip-duration { font-size: 0.8125rem; color: var(--text-secondary); margin: 4px 0; }
  .trip-route-line { display: flex; flex-direction: column; align-items: center; width: 24px; }
  .trip-route-dot { width: 14px; height: 14px; border-radius: 50%; border: 3px solid var(--green); background: var(--white); }
  .trip-route-dot.arrival { border-color: var(--text); }
  .trip-route-bar { width: 3px; flex: 1; background: var(--border); min-height: 40px; }
  .trip-cities { flex: 1; }
  .trip-city { font-size: 1rem; font-weight: 700; color: var(--text); margin-bottom: 24px; }
  .trip-city:last-child { margin-bottom: 0; }
  .trip-price { text-align: right; min-width: 100px; }
  .trip-price-value { font-size: 1.375rem; font-weight: 800; color: var(--text); }
  .trip-no-seats { font-size: 1.25rem; font-weight: 800; color: var(--text-secondary); font-style: italic; }
  .trip-driver { padding: 14px 20px; border-top: 1px solid var(--border); display: flex; align-items: center; gap: 12px; flex-wrap: wrap; }
  .driver-avatar { width: 44px; height: 44px; border-radius: 50%; background: linear-gradient(135deg, #00B87C 0%, #00D68F 100%); display: flex; align-items: center; justify-content: center; font-size: 1.125rem; font-weight: 700; color: white; position: relative; flex-shrink: 0; }
  .driver-avatar.verified::after { content: '✓'; position: absolute; bottom: -2px; right: -2px; width: 18px; height: 18px; background: var(--green); border-radius: 50%; font-size: 0.625rem; display: flex; align-items: center; justify-content: center; color: white; border: 2px solid white; }
  .driver-info { flex: 1; min-width: 0; }
  .driver-name { font-size: 0.9375rem; font-weight: 700; color: var(--text); }
  .driver-rating { font-size: 0.8125rem; color: var(--text-secondary); display: flex; align-items: center; gap: 4px; }
  .driver-rating .star { color: #FFB800; }
  .driver-phone { font-size: 0.875rem; color: var(--green); font-weight: 600; margin-top: 4px; }
  .action-buttons { display: flex; gap: 8px; align-items: center; }
  .call-btn-small, .write-btn-small { padding: 8px 14px; border: none; border-radius: 10px; font-size: 0.8125rem; font-weight: 700; cursor: pointer; font-family: inherit; text-decoration: none; white-space: nowrap; }
  .call-btn-small { background: var(--green); color: white; }
  .call-btn-small:active { background: var(--green-dark); transform: scale(0.95); }
  .write-btn-small { background: var(--green-light); color: var(--green); }
  .write-btn-small:active { background: #D0F0E4; transform: scale(0.95); }

  .calendar-screen-content { padding-bottom: 100px; }
  .calendar-header { padding: 20px 16px; }
  .calendar-title { font-size: 1.75rem; font-weight: 800; color: var(--text); margin-bottom: 24px; }
  .weekdays { display: grid; grid-template-columns: repeat(7, 1fr); text-align: center; margin-bottom: 16px; padding: 0 8px; }
  .weekday { font-size: 0.875rem; font-weight: 600; color: var(--text); padding: 8px 0; }
  .month-title { font-size: 1.375rem; font-weight: 800; color: var(--text); margin: 20px 8px 12px; }
  .calendar-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 4px; padding: 0 8px; }
  .calendar-day { aspect-ratio: 1; display: flex; align-items: center; justify-content: center; border-radius: 12px; font-size: 1rem; font-weight: 600; cursor: pointer; border: 2px solid transparent; min-height: 44px; }
  .calendar-day.empty { cursor: default; }
  .calendar-day.past { color: #B0B8C4; cursor: default; }
  .calendar-day.available { color: var(--text); border: 2px solid var(--border); background: var(--white); }
  .calendar-day.available:active { transform: scale(0.92); }
  .calendar-day.selected { background: var(--green); color: white; border-color: var(--green); }
  .calendar-day.today { border-color: var(--green); color: var(--green); font-weight: 800; }

  .requests-header-info { padding: 16px; }
  .requests-title { font-size: 1.375rem; font-weight: 800; color: var(--text); margin-bottom: 8px; }
  .requests-subtitle { font-size: 0.875rem; color: var(--text-secondary); line-height: 1.4; margin-bottom: 16px; }
  .create-request-btn { display: flex; align-items: center; justify-content: center; gap: 10px; width: calc(100% - 32px); margin: 0 16px 16px; padding: 16px; background: var(--green); color: white; border: none; border-radius: 16px; font-size: 1rem; font-weight: 700; cursor: pointer; font-family: inherit; }
  .create-request-btn:active { background: var(--green-dark); transform: scale(0.98); }
  .request-card { background: var(--white); border-radius: 20px; margin: 0 16px 12px; overflow: hidden; box-shadow: var(--shadow); border-left: 4px solid var(--green); }
  .request-main { padding: 18px; }
  .request-route { display: flex; align-items: center; gap: 12px; margin-bottom: 12px; }
  .request-route-text { flex: 1; }
  .request-from, .request-to { font-size: 1rem; font-weight: 700; color: var(--text); }
  .request-arrow-icon { color: var(--green); flex-shrink: 0; }
  .request-meta { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px; }
  .request-tag { padding: 6px 12px; background: var(--green-light); color: var(--green-dark); border-radius: 20px; font-size: 0.8125rem; font-weight: 600; }
  .request-comment { font-size: 0.875rem; color: var(--text-secondary); line-height: 1.4; margin-bottom: 12px; font-style: italic; }
  .request-footer { padding: 12px 18px; border-top: 1px solid var(--border); display: flex; align-items: center; justify-content: space-between; gap: 10px; flex-wrap: wrap; }
  .request-author { display: flex; align-items: center; gap: 10px; flex: 1; min-width: 0; }
  .request-author-avatar { width: 36px; height: 36px; border-radius: 50%; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); display: flex; align-items: center; justify-content: center; color: white; font-weight: 700; font-size: 0.875rem; flex-shrink: 0; }
  .request-author-name { font-size: 0.875rem; font-weight: 600; color: var(--text); }
  .request-author-date { font-size: 0.75rem; color: var(--text-secondary); }
  .request-actions { display: flex; gap: 8px; }
  .respond-btn { padding: 10px 16px; background: var(--green); color: white; border: none; border-radius: 12px; font-size: 0.875rem; font-weight: 700; cursor: pointer; font-family: inherit; }
  .respond-btn:active { background: var(--green-dark); }

  .ai-fab { position: fixed; bottom: 90px; right: 16px; width: 60px; height: 60px; border-radius: 50%; background: linear-gradient(135deg, #00B87C 0%, #00D68F 100%); color: white; border: none; cursor: pointer; z-index: 140; display: flex; align-items: center; justify-content: center; box-shadow: 0 4px 20px rgba(0, 184, 124, 0.4); animation: pulse 2s infinite; }
  .ai-fab:active { transform: scale(0.9); }
  .ai-modal-overlay { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.6); z-index: 200; align-items: flex-end; justify-content: center; }
  .ai-modal-overlay.active { display: flex; }
  .ai-modal-content { background: var(--white); border-radius: 24px 24px 0 0; width: 100%; max-width: 500px; height: 90vh; display: flex; flex-direction: column; animation: slideUp 0.3s ease; overflow: hidden; }
  .ai-header { padding: 16px 20px; background: linear-gradient(135deg, #00B87C 0%, #00D68F 100%); color: white; display: flex; align-items: center; gap: 12px; }
  .ai-header-avatar { width: 44px; height: 44px; background: rgba(255,255,255,0.2); border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 1.375rem; }
  .ai-header-info { flex: 1; }
  .ai-header-name { font-size: 1.0625rem; font-weight: 800; }
  .ai-header-status { font-size: 0.75rem; opacity: 0.9; display: flex; align-items: center; gap: 4px; }
  .ai-header-status::before { content: ''; width: 8px; height: 8px; background: #00FF99; border-radius: 50%; }
  .ai-close { width: 36px; height: 36px; border-radius: 50%; background: rgba(255,255,255,0.2); border: none; color: white; cursor: pointer; font-size: 1.125rem; display: flex; align-items: center; justify-content: center; }
  .ai-messages { flex: 1; overflow-y: auto; padding: 16px; display: flex; flex-direction: column; gap: 12px; background: var(--bg); }
  .ai-message { max-width: 90%; padding: 14px 18px; border-radius: 18px; font-size: 1.0625rem; line-height: 1.5; animation: fadeIn 0.3s ease; white-space: pre-wrap; }
  .ai-message.bot { background: var(--white); color: var(--text); align-self: flex-start; border-bottom-left-radius: 4px; box-shadow: 0 1px 4px rgba(0,0,0,0.05); }
  .ai-message.user { background: var(--green); color: white; align-self: flex-end; border-bottom-right-radius: 4px; }
  .ai-message .msg-time { font-size: 0.6875rem; opacity: 0.7; margin-top: 4px; display: block; }
  .ai-message .action-btn { display: inline-block; margin: 6px 4px 0 0; padding: 10px 16px; background: var(--green); color: white; border-radius: 12px; text-decoration: none; font-size: 0.875rem; font-weight: 700; cursor: pointer; border: none; font-family: inherit; }
  .ai-message .action-btn:active { background: var(--green-dark); }
  .ai-message .action-btn.secondary { background: var(--green-light); color: var(--green); }
  .ai-quick-replies { display: flex; flex-wrap: wrap; gap: 6px; padding: 0 16px 8px; background: var(--bg); }
  .ai-quick-reply { padding: 10px 16px; background: var(--white); border: 1.5px solid var(--green); color: var(--green); border-radius: 20px; font-size: 0.875rem; font-weight: 600; cursor: pointer; font-family: inherit; }
  .ai-quick-reply:active { background: var(--green-light); }
  .ai-input-area { padding: 12px 16px; background: var(--white); border-top: 1px solid var(--border); display: flex; gap: 8px; align-items: center; }
  .ai-input { flex: 1; padding: 12px 16px; border: 1.5px solid var(--border); border-radius: 24px; font-size: 1rem; font-family: inherit; outline: none; color: var(--text); }
  .ai-input:focus { border-color: var(--green); }
  .ai-voice-btn, .ai-send-btn { width: 48px; height: 48px; border-radius: 50%; border: none; cursor: pointer; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
  .ai-voice-btn { background: var(--green-light); color: var(--green); position: relative; }
  .ai-voice-btn.recording { background: #FF4757; color: white; }
  .ai-voice-btn.recording::before { content: ''; position: absolute; inset: -4px; border-radius: 50%; border: 3px solid #FF4757; animation: ripple 1.2s infinite; }
  .ai-voice-btn.recording::after { content: ''; position: absolute; inset: -12px; border-radius: 50%; border: 2px solid #FF4757; animation: ripple 1.2s infinite 0.3s; }
  .ai-send-btn { background: var(--green); color: white; }
  .ai-typing { display: flex; gap: 4px; padding: 12px 16px; background: var(--white); border-radius: 18px; align-self: flex-start; box-shadow: 0 1px 4px rgba(0,0,0,0.05); }
  .ai-typing span { width: 8px; height: 8px; background: var(--text-secondary); border-radius: 50%; animation: typing 1.4s infinite; }
  .ai-typing span:nth-child(2) { animation-delay: 0.2s; }
  .ai-typing span:nth-child(3) { animation-delay: 0.4s; }

  /* Voice visualization */
  .voice-visualizer { display: none; align-items: center; justify-content: center; gap: 4px; height: 32px; padding: 0 16px; background: var(--bg); }
  .voice-visualizer.active { display: flex; }
  .voice-bar { width: 4px; background: var(--green); border-radius: 2px; animation: wave 0.8s ease-in-out infinite; }
  .voice-bar:nth-child(1) { animation-delay: 0s; }
  .voice-bar:nth-child(2) { animation-delay: 0.1s; }
  .voice-bar:nth-child(3) { animation-delay: 0.2s; }
  .voice-bar:nth-child(4) { animation-delay: 0.3s; }
  .voice-bar:nth-child(5) { animation-delay: 0.4s; }
  .voice-bar:nth-child(6) { animation-delay: 0.3s; }
  .voice-bar:nth-child(7) { animation-delay: 0.2s; }

  .subtitle-bar { position: fixed; bottom: 80px; left: 16px; right: 16px; background: rgba(0,0,0,0.85); color: white; padding: 14px 18px; border-radius: 14px; font-size: 1.0625rem; font-weight: 600; z-index: 180; display: none; animation: subtitleFade 0.3s ease; text-align: center; line-height: 1.4; }
  .subtitle-bar.show { display: block; }
  .subtitle-bar .subtitle-label { font-size: 0.6875rem; opacity: 0.7; margin-bottom: 4px; text-transform: uppercase; letter-spacing: 0.5px; }

  .access-modal-overlay { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.6); z-index: 250; align-items: flex-end; justify-content: center; }
  .access-modal-overlay.active { display: flex; }
  .access-modal-content { background: var(--white); border-radius: 24px 24px 0 0; width: 100%; max-width: 500px; padding: 20px 16px; animation: slideUp 0.3s ease; }
  .access-modal-handle { width: 40px; height: 4px; background: var(--border); border-radius: 4px; margin: 0 auto 16px; }
  .access-modal-title { font-size: 1.25rem; font-weight: 800; color: var(--text); margin-bottom: 20px; text-align: center; }
  .access-option { display: flex; align-items: center; justify-content: space-between; padding: 16px; background: var(--bg); border-radius: 14px; margin-bottom: 10px; }
  .access-option-info { flex: 1; }
  .access-option-title { font-size: 0.9375rem; font-weight: 700; color: var(--text); margin-bottom: 2px; }
  .access-option-desc { font-size: 0.75rem; color: var(--text-secondary); }
  .access-toggle { width: 50px; height: 28px; background: var(--border); border-radius: 14px; position: relative; cursor: pointer; transition: background 0.3s; flex-shrink: 0; margin-left: 12px; }
  .access-toggle.active { background: var(--green); }
  .access-toggle::after { content: ''; position: absolute; top: 3px; left: 3px; width: 22px; height: 22px; background: white; border-radius: 50%; transition: transform 0.3s; box-shadow: 0 1px 4px rgba(0,0,0,0.2); }
  .access-toggle.active::after { transform: translateX(22px); }
  .access-font-sizes { display: flex; gap: 8px; margin-top: 8px; }
  .access-font-btn { flex: 1; padding: 10px; background: var(--white); border: 2px solid var(--border); border-radius: 10px; cursor: pointer; font-family: inherit; font-weight: 700; color: var(--text); }
  .access-font-btn.active { border-color: var(--green); background: var(--green-light); color: var(--green); }

  .bottom-nav { position: fixed; bottom: 0; left: 0; right: 0; background: var(--white); display: flex; border-top: 1px solid var(--border); z-index: 100; padding-bottom: env(safe-area-inset-bottom, 0px); }
  .nav-item { flex: 1; display: flex; flex-direction: column; align-items: center; padding: 10px 4px 8px; cursor: pointer; }
  .nav-item.active .nav-icon { color: var(--green); }
  .nav-item.active .nav-label { color: var(--green); }
  .nav-icon { width: 24px; height: 24px; margin-bottom: 4px; color: var(--text-secondary); }
  .nav-label { font-size: 0.6875rem; font-weight: 600; color: var(--text-secondary); }

  .toast { position: fixed; top: 60px; left: 16px; right: 16px; background: var(--text); color: white; padding: 14px 20px; border-radius: 14px; font-size: 0.875rem; font-weight: 600; z-index: 400; transform: translateY(-100px); opacity: 0; transition: all 0.3s ease; text-align: center; }
  .toast.show { transform: translateY(0); opacity: 1; }

  .publish-form { padding: 16px; }
  .publish-form .form-group { background: var(--white); border-radius: 14px; padding: 14px 16px; margin-bottom: 10px; border: 1px solid var(--border); }
  .publish-form .form-label { font-size: 0.75rem; color: var(--text-secondary); margin-bottom: 6px; }
  .publish-form .form-input { width: 100%; border: none; outline: none; font-size: 1rem; font-family: inherit; font-weight: 600; color: var(--text); background: transparent; }
  .publish-form .form-input::placeholder { color: var(--text-secondary); font-weight: 400; }
  .publish-form .form-input-field { width: 100%; border: none; outline: none; font-size: 1rem; font-family: inherit; font-weight: 600; color: var(--text); background: transparent; padding: 0; }
  .publish-form select { width: 100%; padding: 12px; border: 1.5px solid var(--border); border-radius: 12px; font-size: 1rem; font-family: inherit; outline: none; background: white; color: var(--text); }

  .profile-header { background: var(--white); padding: 30px 16px 20px; text-align: center; }
  .profile-avatar { width: 80px; height: 80px; border-radius: 50%; background: linear-gradient(135deg, #00B87C 0%, #00D68F 100%); display: flex; align-items: center; justify-content: center; font-size: 2rem; font-weight: 700; color: white; margin: 0 auto 12px; border: 3px solid var(--green); }
  .profile-name { font-size: 1.375rem; font-weight: 800; color: var(--text); margin-bottom: 4px; }
  .profile-rating { font-size: 0.875rem; color: var(--text-secondary); display: flex; align-items: center; justify-content: center; gap: 4px; }
  .profile-stats { display: flex; background: var(--white); margin: 16px; border-radius: 16px; overflow: hidden; box-shadow: var(--shadow); }
  .profile-stat { flex: 1; padding: 16px 8px; text-align: center; border-right: 1px solid var(--border); }
  .profile-stat:last-child { border-right: none; }
  .profile-stat-value { font-size: 1.25rem; font-weight: 800; color: var(--text); }
  .profile-stat-label { font-size: 0.75rem; color: var(--text-secondary); margin-top: 4px; }
  .profile-menu { padding: 0 16px; }
  .profile-menu-item { display: flex; align-items: center; gap: 14px; padding: 16px; background: var(--white); border-radius: 14px; margin-bottom: 8px; cursor: pointer; }
  .profile-menu-item:active { background: var(--bg); }
  .profile-menu-icon { width: 36px; height: 36px; display: flex; align-items: center; justify-content: center; background: var(--green-light); border-radius: 10px; color: var(--green); }
  .profile-menu-text { flex: 1; font-size: 0.9375rem; font-weight: 600; color: var(--text); }
  .profile-menu-arrow { color: var(--text-secondary); }

  .map-modal-overlay { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.6); z-index: 300; align-items: center; justify-content: center; padding: 16px; }
  .map-modal-overlay.active { display: flex; }
  .map-modal-content { background: var(--white); border-radius: 20px; width: 100%; max-width: 500px; height: 80vh; overflow: hidden; display: flex; flex-direction: column; animation: slideUp 0.3s ease; }
  .map-modal-header { padding: 16px 20px; border-bottom: 1px solid var(--border); display: flex; align-items: center; justify-content: space-between; }
  .map-modal-title { font-size: 1.125rem; font-weight: 800; color: var(--text); }
  .map-modal-close { width: 32px; height: 32px; border-radius: 50%; background: var(--bg); border: none; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 1.125rem; color: var(--text-secondary); }
  .map-container { flex: 1; position: relative; }
  #map, #siberiaMap { width: 100%; height: 100%; }
  .map-info-bar { padding: 12px 16px; background: var(--green-light); border-top: 1px solid var(--border); font-size: 0.8125rem; color: var(--green-dark); font-weight: 600; text-align: center; }

  .bottom-actions { position: fixed; bottom: 0; left: 0; right: 0; background: var(--white); padding: 12px 16px; display: flex; gap: 12px; box-shadow: 0 -2px 12px rgba(0,0,0,0.08); z-index: 100; }
  .btn-outline { flex: 1; padding: 16px; background: var(--white); color: var(--green); border: 2px solid var(--border); border-radius: 16px; font-size: 1rem; font-weight: 700; cursor: pointer; font-family: inherit; }
  .btn-primary { flex: 1.5; padding: 16px; background: var(--green); color: white; border: none; border-radius: 16px; font-size: 1rem; font-weight: 700; cursor: pointer; font-family: inherit; }
  .btn-primary:active { background: var(--green-dark); transform: scale(0.98); }

  ::-webkit-scrollbar { width: 0; height: 0; }
  .no-results { text-align: center; padding: 20px; color: var(--text-secondary); font-size: 0.875rem; }
  .hidden { display: none !important; }

  .region-map-section { padding: 0 16px; margin-bottom: 20px; }
  .region-map-title { font-size: 1.25rem; font-weight: 800; color: var(--text); margin-bottom: 12px; }
  .region-map-subtitle { font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 14px; line-height: 1.4; }
  .region-map-container { width: 100%; height: 280px; border-radius: 16px; overflow: hidden; box-shadow: var(--shadow); border: 1px solid var(--border); }

  /* Voice Assistant UI */
  .voice-main-btn { width: 88px; height: 88px; border-radius: 50%; background: linear-gradient(135deg, #00B87C 0%, #00D68F 100%); border: none; color: white; cursor: pointer; display: flex; align-items: center; justify-content: center; box-shadow: 0 6px 24px rgba(0, 184, 124, 0.4); position: relative; margin: 16px auto; transition: all 0.3s; }
  .voice-main-btn:active { transform: scale(0.92); }
  .voice-main-btn.recording { background: linear-gradient(135deg, #FF4757 0%, #FF6B7A 100%); box-shadow: 0 6px 24px rgba(255, 71, 87, 0.4); }
  .voice-main-btn.recording::before { content: ''; position: absolute; inset: -6px; border-radius: 50%; border: 3px solid #FF4757; animation: ripple 1.2s infinite; }
  .voice-main-btn.recording::after { content: ''; position: absolute; inset: -16px; border-radius: 50%; border: 2px solid #FF4757; animation: ripple 1.2s infinite 0.3s; }

  .voice-status { text-align: center; padding: 12px; font-size: 0.9375rem; color: var(--text-secondary); font-weight: 600; }
  .voice-status.listening { color: #FF4757; }

  .speed-control { display: flex; align-items: center; gap: 8px; padding: 8px 16px; background: var(--bg); border-radius: 12px; margin: 8px 16px; }
  .speed-control label { font-size: 0.8125rem; color: var(--text-secondary); font-weight: 600; }
  .speed-control select { flex: 1; padding: 6px; border: 1px solid var(--border); border-radius: 8px; font-size: 0.875rem; font-family: inherit; background: white; color: var(--text); }
</style>
</head>
<body>

<button class="accessibility-btn" onclick="openAccessModal()" aria-label="Настройки доступности">
  <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
    <circle cx="12" cy="4" r="2"/><path d="M12 8v6M8 10h8M12 18v4M8 22l4-4 4 4"/>
  </svg>
</button>

<div class="toast" id="toast"></div>

<div class="subtitle-bar" id="subtitleBar">
  <div class="subtitle-label">📝 Субтитры</div>
  <div id="subtitleText"></div>
</div>

<!-- HOME -->
<div class="screen active" id="screen-home">
  <div class="main-header">
    <div class="greeting">
      <svg class="greeting-icon" viewBox="0 0 40 40" fill="none">
        <circle cx="20" cy="20" r="18" fill="#00B87C"/>
        <path d="M12 20c0-4.4 3.6-8 8-8s8 3.6 8 8" stroke="white" stroke-width="2.5" stroke-linecap="round"/>
        <circle cx="14" cy="18" r="2" fill="white"/>
        <circle cx="26" cy="18" r="2" fill="white"/>
        <path d="M15 24c1.5 2 3.5 3 5 3s3.5-1 5-3" stroke="white" stroke-width="2" stroke-linecap="round"/>
      </svg>
      <h1>Здравствуйте, Андрей!</h1>
    </div>
    <div class="main-title">Автобусы и поездки с попутчиками: ваше путешествие — ваш выбор</div>
  </div>

  <div class="type-toggle" style="margin: 0 16px 16px;">
    <div class="type-toggle-btn active" id="togglePassengers" onclick="switchSearchType('passengers')">👥 Пассажиры</div>
    <div class="type-toggle-btn" id="toggleParcels" onclick="switchSearchType('parcels')">📦 Посылки</div>
  </div>

  <div class="search-form">
    <div class="form-group autocomplete-container">
      <div class="form-label" id="fromLabel">Откуда</div>
      <input class="form-input-field" id="homeFromInput" placeholder="Город, вокзал, заведение" autocomplete="off">
      <div class="autocomplete-list" id="fromAutocomplete"></div>
    </div>
    <div class="form-group autocomplete-container">
      <div class="form-label" id="toLabel">Куда</div>
      <input class="form-input-field" id="homeToInput" placeholder="Город, вокзал, заведение" autocomplete="off">
      <div class="autocomplete-list" id="toAutocomplete"></div>
    </div>
    <div class="form-group" onclick="showScreen('screen-calendar')">
      <div class="form-label">Отправление</div>
      <div class="form-value" id="homeDate">Сегодня</div>
    </div>
    <div class="form-group" id="passengersField" onclick="showPassengerModal()">
      <div class="form-label">Пассажиры</div>
      <div class="form-value" id="homePassengers">1 пассажир</div>
    </div>
    <div class="form-group hidden" id="cargoField">
      <div class="form-label">Тип груза</div>
      <select id="homeCargoSelect" onchange="updateCargoDisplay()" style="width: 100%; padding: 8px; border: none; font-size: 1rem; font-family: inherit; font-weight: 600; color: var(--text); background: transparent; outline: none;">
        <option>Документы</option><option>Коробка</option><option>Хрупкое</option><option>Одежда</option><option>Электроника</option><option>Продукты</option><option value="custom">Другое (указать)</option>
      </select>
    </div>
    <div class="form-group hidden" id="customCargoField">
      <div class="form-label">Укажите тип груза</div>
      <input class="form-input-field" id="homeCustomCargo" placeholder="Например: одежда, электроника...">
    </div>
    <button class="search-btn" onclick="doSearch()">Поиск</button>
  </div>

  <div class="region-map-section">
    <div class="region-map-title">🗺️ Ближайшие регионы</div>
    <div class="region-map-subtitle">Кемеровская область, Омская область, Новосибирская область и соседние города.</div>
    <div class="region-map-container"><div id="siberiaMap"></div></div>
  </div>

  <div class="directory-section">
    <div class="directory-header">
      <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#00B87C" stroke-width="2"><path d="M22 16.92v3a2 2 0 01-2.18 2 19.79 19.79 0 01-8.63-3.07 19.5 19.5 0 01-6-6 19.79 19.79 0 01-3.07-8.67A2 2 0 014.11 2h3a2 2 0 012 1.72 12.84 12.84 0 00.7 2.81 2 2 0 01-.45 2.11L8.09 9.91a16 16 0 006 6l1.27-1.27a2 2 0 012.11-.45 12.84 12.84 0 002.81.7A2 2 0 0122 16.92z"/></svg>
      <div class="directory-title">Справочник автовокзалов</div>
    </div>
    <div class="directory-subtitle">Если не нашли подходящую поездку — позвоните на автовокзал.</div>
    <a href="tel:88005553535" class="hotline-banner">
      <div class="hotline-icon"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2.5"><path d="M22 16.92v3a2 2 0 01-2.18 2 19.79 19.79 0 01-8.63-3.07 19.5 19.5 0 01-6-6 19.79 19.79 0 01-3.07-8.67A2 2 0 014.11 2h3a2 2 0 012 1.72 12.84 12.84 0 00.7 2.81 2 2 0 01-.45 2.11L8.09 9.91a16 16 0 006 6l1.27-1.27a2 2 0 012.11-.45 12.84 12.84 0 002.81.7A2 2 0 0122 16.92z"/></svg></div>
      <div class="hotline-content"><div class="hotline-title">Единая справочная</div><div class="hotline-number">8 (800) 555-35-35</div></div>
    </a>
    <input type="text" class="directory-search" id="directorySearch" placeholder="🔍 Найти город...">
    <div class="accordion" id="directoryAccordion"></div>
  </div>

  <div class="bottom-nav">
    <div class="nav-item active" onclick="showScreen('screen-home')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="7"/><path d="M21 21l-4.35-4.35"/></svg><span class="nav-label">Искать</span></div>
    <div class="nav-item" onclick="showScreen('screen-publish')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="9"/><path d="M12 8v8M8 12h8"/></svg><span class="nav-label">Опубликовать</span></div>
    <div class="nav-item" onclick="showScreen('screen-requests')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 00-3-3.87M16 3.13a4 4 0 010 7.75"/></svg><span class="nav-label">Заявки</span></div>
    <div class="nav-item" onclick="showScreen('screen-trips')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 17H5a2 2 0 01-2-2V5a2 2 0 012-2h4M15 7h4a2 2 0 012 2v10a2 2 0 01-2 2h-4"/><circle cx="9" cy="17" r="2"/><circle cx="15" cy="7" r="2"/></svg><span class="nav-label">Поездки</span></div>
    <div class="nav-item" onclick="showScreen('screen-profile')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/><circle cx="12" cy="7" r="4"/></svg><span class="nav-label">Профиль</span></div>
  </div>
</div>

<!-- CALENDAR -->
<div class="screen" id="screen-calendar">
  <div class="header">
    <div class="header-back" onclick="showScreen('screen-home')"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#00B87C" stroke-width="2.5" stroke-linecap="round"><path d="M18 6L6 18M6 6l12 12"/></svg></div>
  </div>
  <div class="calendar-screen-content">
    <div class="calendar-header"><div class="calendar-title">Когда вы едете?</div></div>
    <div class="weekdays"><div class="weekday">Пн</div><div class="weekday">Вт</div><div class="weekday">Ср</div><div class="weekday">Чт</div><div class="weekday">Пт</div><div class="weekday">Сб</div><div class="weekday">Вс</div></div>
    <div class="month-title">Ноября</div><div class="calendar-grid" id="novGrid"></div>
    <div class="month-title">Декабря</div><div class="calendar-grid" id="decGrid"></div>
  </div>
  <div class="bottom-nav">
    <div class="nav-item" onclick="showScreen('screen-home')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="7"/><path d="M21 21l-4.35-4.35"/></svg><span class="nav-label">Искать</span></div>
    <div class="nav-item" onclick="showScreen('screen-publish')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="9"/><path d="M12 8v8M8 12h8"/></svg><span class="nav-label">Опубликовать</span></div>
    <div class="nav-item" onclick="showScreen('screen-requests')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 00-3-3.87M16 3.13a4 4 0 010 7.75"/></svg><span class="nav-label">Заявки</span></div>
    <div class="nav-item" onclick="showScreen('screen-trips')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 17H5a2 2 0 01-2-2V5a2 2 0 012-2h4M15 7h4a2 2 0 012 2v10a2 2 0 01-2 2h-4"/><circle cx="9" cy="17" r="2"/><circle cx="15" cy="7" r="2"/></svg><span class="nav-label">Поездки</span></div>
    <div class="nav-item" onclick="showScreen('screen-profile')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/><circle cx="12" cy="7" r="4"/></svg><span class="nav-label">Профиль</span></div>
  </div>
</div>

<!-- RESULTS -->
<div class="screen" id="screen-results">
  <div class="header">
    <div class="header-back" onclick="showScreen('screen-home')"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#00B87C" stroke-width="2.5" stroke-linecap="round"><path d="M19 12H5M12 19l-7-7 7-7"/></svg></div>
  </div>
  <div class="route-header" onclick="showScreen('screen-home')" style="background: var(--white); padding: 16px; border-radius: 16px; margin: 16px; box-shadow: var(--shadow); display: flex; align-items: center; gap: 10px; cursor: pointer;">
    <svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="#6B7C93" stroke-width="2"><path d="M15 10H5M5 10l4-4M5 10l4 4"/></svg>
    <div style="flex: 1;"><div style="font-size: 0.875rem; font-weight: 700; color: var(--text);" id="resultRoute">Новосибирск → Славгород</div><div style="font-size: 0.8125rem; color: var(--text-secondary);" id="resultDate">Чт 19 нояб., 1 пассажир</div></div>
  </div>
  <div class="tabs">
    <div class="tab active"><div class="tab-label">Всё</div><div class="tab-count">2</div></div>
    <div class="tab"><div class="tab-label">С попутчиками</div><div class="tab-count">2</div></div>
    <div class="tab"><div class="tab-label">На автобусе</div><div class="tab-count">0</div></div>
  </div>
  <div class="alert-banner" style="margin: 16px; padding: 16px; background: var(--white); border-radius: 16px; display: flex; gap: 14px; align-items: flex-start; box-shadow: var(--shadow);">
    <div style="width: 48px; height: 48px; flex-shrink: 0; display: flex; align-items: center; justify-content: center;"><svg width="48" height="48" viewBox="0 0 48 48" fill="none"><path d="M24 8c-6 0-11 4-12 10l-2 10c0 2 1 4 3 4h22c2 0 3-2 3-4l-2-10c-1-6-6-10-12-10z" fill="#00B87C" opacity="0.3"/><circle cx="24" cy="38" r="4" fill="#00B87C"/></svg></div>
    <div style="flex: 1;"><h3 style="font-size: 0.9375rem; font-weight: 700; color: var(--text); margin-bottom: 4px;">Популярный маршрут — лучше не затягивать с бронью.</h3><p style="font-size: 0.875rem; color: var(--text-secondary);">50% поездок уже забронировано.</p></div>
  </div>
  <div class="trip-card disabled">
    <div class="trip-main">
      <div class="trip-times"><div class="trip-departure">07:00</div><div class="trip-duration">7 ч 40</div><div class="trip-arrival">14:40</div></div>
      <div class="trip-route-line"><div class="trip-route-dot"></div><div class="trip-route-bar"></div><div class="trip-route-dot arrival"></div></div>
      <div class="trip-cities"><div class="trip-city">Новосибирск</div><div class="trip-city">Славгород</div></div>
      <div class="trip-price"><div class="trip-no-seats">Мест нет</div></div>
    </div>
  </div>
  <div class="trip-card" onclick="showScreen('screen-detail')">
    <div class="trip-main">
      <div class="trip-times"><div class="trip-departure">17:00</div><div class="trip-duration">5 ч 30</div><div class="trip-arrival">22:30</div></div>
      <div class="trip-route-line"><div class="trip-route-dot"></div><div class="trip-route-bar"></div><div class="trip-route-dot arrival"></div></div>
      <div class="trip-cities"><div class="trip-city">Новосибирск</div><div class="trip-city">Славгород</div></div>
      <div class="trip-price"><div class="trip-price-value">1 850<sup>,00 ₽</sup></div></div>
    </div>
    <div class="trip-driver"><div class="driver-avatar verified">В</div><div class="driver-info"><div class="driver-name">Василий</div><div class="driver-rating"><span class="star">★</span> 5</div><a href="tel:+79991234567" style="font-size: 0.875rem; color: var(--green); font-weight: 600; text-decoration: none;">📞 +7 (999) 123-45-67</a></div><div class="action-buttons"><a href="tel:+79991234567" class="call-btn-small">📞</a><button class="write-btn-small" onclick="event.stopPropagation(); openChat('Василий')">✉️</button></div></div>
  </div>
  <button class="search-btn" style="margin: 20px 16px; background: var(--white); color: var(--green); border: 2px solid var(--green);" onclick="showToast('Уведомление создано!')">Создать уведомление</button>
  <div class="bottom-nav">
    <div class="nav-item" onclick="showScreen('screen-home')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="7"/><path d="M21 21l-4.35-4.35"/></svg><span class="nav-label">Искать</span></div>
    <div class="nav-item" onclick="showScreen('screen-publish')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="9"/><path d="M12 8v8M8 12h8"/></svg><span class="nav-label">Опубликовать</span></div>
    <div class="nav-item" onclick="showScreen('screen-requests')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 00-3-3.87M16 3.13a4 4 0 010 7.75"/></svg><span class="nav-label">Заявки</span></div>
    <div class="nav-item" onclick="showScreen('screen-trips')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 17H5a2 2 0 01-2-2V5a2 2 0 012-2h4M15 7h4a2 2 0 012 2v10a2 2 0 01-2 2h-4"/><circle cx="9" cy="17" r="2"/><circle cx="15" cy="7" r="2"/></svg><span class="nav-label">Поездки</span></div>
    <div class="nav-item" onclick="showScreen('screen-profile')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/><circle cx="12" cy="7" r="4"/></svg><span class="nav-label">Профиль</span></div>
  </div>
</div>

<!-- DETAIL -->
<div class="screen" id="screen-detail">
  <div class="header">
    <div class="header-back" onclick="showScreen('screen-results')"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#00B87C" stroke-width="2.5" stroke-linecap="round"><path d="M19 12H5M12 19l-7-7 7-7"/></svg></div>
  </div>
  <div style="padding: 20px 16px;">
    <div style="font-size: 1.625rem; font-weight: 800; color: var(--text); margin-bottom: 20px;">Четверг, 19 ноября</div>
    <div style="display: flex; gap: 16px; align-items: flex-start;">
      <div style="display: flex; flex-direction: column; align-items: flex-end; min-width: 60px; padding-top: 4px;">
        <div style="font-size: 1.375rem; font-weight: 800; color: var(--text);">17:00</div>
        <div style="font-size: 0.8125rem; color: var(--text-secondary); margin: 8px 0;">5 ч 30</div>
        <div style="font-size: 1.375rem; font-weight: 800; color: var(--text);">22:30</div>
      </div>
      <div style="display: flex; flex-direction: column; align-items: center; width: 24px;">
        <div style="width: 16px; height: 16px; border-radius: 50%; border: 3px solid var(--green); background: var(--white);"></div>
        <div style="width: 3px; flex: 1; background: var(--text); min-height: 50px;"></div>
        <div style="width: 16px; height: 16px; border-radius: 50%; border: 3px solid var(--text); background: var(--white);"></div>
      </div>
      <div style="flex: 1;">
        <div style="font-size: 1.125rem; font-weight: 800; color: var(--text); display: flex; align-items: center; gap: 8px; cursor: pointer;" onclick="openMap('departure')">Новосибирск <svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="#00B87C" stroke-width="1.5"><path d="M10 2C6.5 2 4 5 4 8c0 4 6 10 6 10s6-6 6-10c0-3-2.5-6-6-6z"/><circle cx="10" cy="8" r="2"/></svg></div>
        <div style="font-size: 0.875rem; color: var(--text-secondary); margin-top: 4px; margin-bottom: 20px; cursor: pointer;" onclick="openMap('departure')">Ул. Шамшурина, 43А</div>
        <div style="font-size: 1.125rem; font-weight: 800; color: var(--text); display: flex; align-items: center; gap: 8px; cursor: pointer;" onclick="openMap('arrival')">Славгород <svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="#00B87C" stroke-width="1.5"><path d="M10 2C6.5 2 4 5 4 8c0 4 6 10 6 10s6-6 6-10c0-3-2.5-6-6-6z"/><circle cx="10" cy="8" r="2"/></svg></div>
        <div style="font-size: 0.875rem; color: var(--text-secondary); margin-top: 4px; cursor: pointer;" onclick="openMap('arrival')">Ул. Ленина, 81</div>
      </div>
    </div>
  </div>
  <div style="background: var(--white); padding: 16px 20px; display: flex; justify-content: space-between; align-items: center; margin: 16px; border-radius: 16px; box-shadow: var(--shadow);">
    <div style="font-size: 1rem; font-weight: 700; color: var(--text);">1 пассажир</div>
    <div style="font-size: 1.5rem; font-weight: 800; color: var(--text);">1 850<sup>,00</sup> ₽</div>
  </div>
  <div style="background: var(--white); margin: 0 16px 16px; border-radius: 20px; overflow: hidden; box-shadow: var(--shadow);">
    <div style="padding: 20px; display: flex; align-items: center; gap: 14px;">
      <div style="width: 56px; height: 56px; border-radius: 50%; background: linear-gradient(135deg, #00B87C 0%, #00D68F 100%); display: flex; align-items: center; justify-content: center; font-size: 1.375rem; font-weight: 700; color: white; position: relative; border: 3px solid var(--green);">В<div style="position: absolute; bottom: 0; right: 0; width: 20px; height: 20px; background: var(--green); border-radius: 50%; font-size: 0.6875rem; display: flex; align-items: center; justify-content: center; color: white; border: 2px solid white;">✓</div></div>
      <div style="flex: 1;"><div style="font-size: 1.125rem; font-weight: 800; color: var(--text); margin-bottom: 4px;">Василий</div><div style="font-size: 0.875rem; color: var(--text-secondary); display: flex; align-items: center; gap: 4px;"><span style="color: var(--green); font-size: 1rem;">★</span> 5/5 - 102 отзыва</div><a href="tel:+79991234567" style="font-size: 0.875rem; color: var(--green); font-weight: 600; margin-top: 4px; text-decoration: none; display: inline-block;">📞 +7 (999) 123-45-67</a></div>
    </div>
    <div style="padding: 0 20px 20px;">
      <div style="display: flex; align-items: center; gap: 14px; padding: 14px 0; border-bottom: 1px solid var(--border);"><div style="width: 28px; height: 28px; display: flex; align-items: center; justify-content: center; flex-shrink: 0;"><svg width="28" height="28" viewBox="0 0 28 28" fill="none"><path d="M14 3l3 6 7 1-5 5 1 7-6-3-6 3 1-7-5-5 7-1z" fill="#00B87C"/><path d="M10 14l3 3 5-6" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg></div><div style="font-size: 0.9375rem; color: var(--text); font-weight: 500;">Профиль подтвержден</div></div>
      <div style="display: flex; align-items: center; gap: 14px; padding: 14px 0; border-bottom: 1px solid var(--border);"><div style="width: 28px; height: 28px; display: flex; align-items: center; justify-content: center; flex-shrink: 0;"><svg width="28" height="28" viewBox="0 0 28 28" fill="none" stroke="#00B87C" stroke-width="1.5"><path d="M14 4v4M14 20v4M4 14h4M20 14h4M7 7l3 3M18 18l3 3M7 21l3-3M18 10l3-3"/><circle cx="14" cy="14" r="4"/></svg></div><div style="font-size: 0.9375rem; color: var(--text); font-weight: 500;">Кондиционер</div></div>
      <div style="display: flex; align-items: center; gap: 14px; padding: 14px 0;"><div style="width: 28px; height: 28px; display: flex; align-items: center; justify-content: center; flex-shrink: 0;"><svg width="28" height="28" viewBox="0 0 28 28" fill="none" stroke="#6B7C93" stroke-width="1.5"><path d="M5 18h18M5 18a2 2 0 01-2-2V10a2 2 0 012-2h2l2-3h8l2 3h2a2 2 0 012 2v6a2 2 0 01-2 2M5 18v2a1 1 0 001 1h2a1 1 0 001-1v-2m10 0v2a1 1 0 001 1h2a1 1 0 001-1v-2"/><circle cx="8" cy="15" r="1.5"/><circle cx="20" cy="15" r="1.5"/></svg></div><div style="font-size: 0.9375rem; color: var(--text); font-weight: 500;">TOYOTA Premio - Серый</div></div>
    </div>
    <div style="padding: 0 20px 20px; display: flex; gap: 10px;">
      <a href="tel:+79991234567" class="call-btn" style="flex: 1;">📞 Позвонить</a>
      <button class="call-btn secondary" style="flex: 1;" onclick="openChat('Василий')">✉️ Написать</button>
    </div>
  </div>
  <div style="height: 80px;"></div>
  <div class="bottom-actions">
    <button class="btn-outline" onclick="openChat('Василий')">Написать</button>
    <button class="btn-primary" onclick="bookTrip()">Забронировать</button>
  </div>
</div>

<!-- PUBLISH -->
<div class="screen" id="screen-publish">
  <div class="header">
    <div class="header-back" onclick="showScreen('screen-home')"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#00B87C" stroke-width="2.5" stroke-linecap="round"><path d="M19 12H5M12 19l-7-7 7-7"/></svg></div>
    <div class="header-title">Опубликовать поездку</div>
  </div>
  <div class="type-toggle" style="margin: 16px;">
    <div class="type-toggle-btn active" id="pubTogglePassengers" onclick="switchPublishType('passengers')">👥 Беру попутчиков</div>
    <div class="type-toggle-btn" id="pubToggleParcels" onclick="switchPublishType('parcels')">📦 Беру посылки</div>
  </div>
  <div class="publish-form">
    <div class="form-group autocomplete-container"><div class="form-label">Откуда</div><input class="form-input-field" id="pubFromInput" placeholder="Город отправления" autocomplete="off"><div class="autocomplete-list" id="pubFromAutocomplete"></div></div>
    <div class="form-group autocomplete-container"><div class="form-label">Куда</div><input class="form-input-field" id="pubToInput" placeholder="Город назначения" autocomplete="off"><div class="autocomplete-list" id="pubToAutocomplete"></div></div>
    <div class="form-group" onclick="showScreen('screen-calendar')"><div class="form-label">Дата отправления</div><div class="form-value" id="pubDate">Выбрать дату</div></div>
    <div class="form-group"><div class="form-label">Время отправления</div><input class="form-input-field" type="time" value="09:00"></div>
    <div id="pubPassengerFields">
      <div class="form-group"><div class="form-label">Свободных мест</div><input class="form-input-field" type="number" value="3" min="1" max="7"></div>
      <div class="form-group"><div class="form-label">Цена за место (₽)</div><input class="form-input-field" type="number" value="1500" min="0"></div>
    </div>
    <div id="pubParcelFields" class="hidden">
      <div class="form-group">
        <div class="form-label">Тип груза</div>
        <select id="pubCargoType" onchange="togglePubCustomCargo()">
          <option>Документы</option><option>Коробка</option><option>Хрупкое</option><option>Одежда</option><option>Электроника</option><option>Продукты</option><option value="custom">Другое (указать вручную)</option>
        </select>
      </div>
      <div class="form-group hidden" id="pubCustomCargoField">
        <div class="form-label">Укажите тип груза</div>
        <input class="form-input-field" id="pubCustomCargo" placeholder="Например: одежда, электроника, продукты...">
      </div>
      <div class="form-group"><div class="form-label">Максимальный вес (кг)</div><input class="form-input-field" type="number" id="pubWeight" placeholder="Например: 10" min="1" max="15"></div>
      <div class="form-group"><div class="form-label">Габариты (Д×Ш×В, см)</div><input class="form-input-field" type="text" id="pubDimensions" placeholder="Например: 40×30×20"></div>
      <div class="form-group"><div class="form-label">Стоимость доставки (₽)</div><input class="form-input-field" type="number" id="pubParcelPrice" placeholder="Например: 500" min="1"></div>
    </div>
    <div class="form-group"><div class="form-label">Автомобиль</div><input class="form-input-field" placeholder="Марка и модель"></div>
    <button class="search-btn" onclick="publishTrip()">Опубликовать поездку</button>
  </div>
  <div class="bottom-nav">
    <div class="nav-item" onclick="showScreen('screen-home')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="7"/><path d="M21 21l-4.35-4.35"/></svg><span class="nav-label">Искать</span></div>
    <div class="nav-item active" onclick="showScreen('screen-publish')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="9"/><path d="M12 8v8M8 12h8"/></svg><span class="nav-label">Опубликовать</span></div>
    <div class="nav-item" onclick="showScreen('screen-requests')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 00-3-3.87M16 3.13a4 4 0 010 7.75"/></svg><span class="nav-label">Заявки</span></div>
    <div class="nav-item" onclick="showScreen('screen-trips')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 17H5a2 2 0 01-2-2V5a2 2 0 012-2h4M15 7h4a2 2 0 012 2v10a2 2 0 01-2 2h-4"/><circle cx="9" cy="17" r="2"/><circle cx="15" cy="7" r="2"/></svg><span class="nav-label">Поездки</span></div>
    <div class="nav-item" onclick="showScreen('screen-profile')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/><circle cx="12" cy="7" r="4"/></svg><span class="nav-label">Профиль</span></div>
  </div>
</div>

<!-- REQUESTS -->
<div class="screen" id="screen-requests">
  <div class="header">
    <div class="header-back" onclick="showScreen('screen-home')"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#00B87C" stroke-width="2.5" stroke-linecap="round"><path d="M19 12H5M12 19l-7-7 7-7"/></svg></div>
    <div class="header-title">Заявки пассажиров</div>
  </div>
  <div class="requests-header-info">
    <div class="requests-title">🙋 Пассажиры ищут поездку</div>
    <div class="requests-subtitle">Здесь пассажиры оставляют заявки на определённую дату. Если вы водитель и можете подвезти — откликнитесь!</div>
  </div>
  <button class="create-request-btn" onclick="openRequestModal()">
    <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><circle cx="12" cy="12" r="9"/><path d="M12 8v8M8 12h8"/></svg>
    Оставить заявку
  </button>
  <div class="tabs" style="margin-bottom: 16px;">
    <div class="tab active" onclick="filterRequests(this, 'all')"><div class="tab-label">Все</div><div class="tab-count">4</div></div>
    <div class="tab" onclick="filterRequests(this, 'today')"><div class="tab-label">Сегодня</div><div class="tab-count">1</div></div>
    <div class="tab" onclick="filterRequests(this, 'week')"><div class="tab-label">Неделя</div><div class="tab-count">3</div></div>
  </div>
  <div id="requestsList">
    <div class="request-card">
      <div class="request-main">
        <div class="request-route"><div class="request-route-text"><div class="request-from">Новосибирск</div><div style="display: flex; align-items: center; gap: 6px; margin: 4px 0;"><svg class="request-arrow-icon" width="16" height="16" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2"><path d="M5 10h10M10 5l5 5-5 5"/></svg></div><div class="request-to">Славгород</div></div></div>
        <div class="request-meta"><div class="request-tag">📅 19 ноября</div><div class="request-tag">👥 1 место</div><div class="request-tag">⏰ Утро</div></div>
        <div class="request-comment">«Нужно выехать утром, желательно до 10:00. Багаж — одна сумка.»</div>
      </div>
      <div class="request-footer">
        <div class="request-author"><div class="request-author-avatar">М</div><div><div class="request-author-name">Мария К.</div><div class="request-author-date">2 часа назад</div><a href="tel:+79992345678" style="font-size: 0.8125rem; color: var(--green); font-weight: 600; text-decoration: none;">📞 +7 (999) 234-56-78</a></div></div>
        <div class="request-actions">
          <a href="tel:+79992345678" class="call-btn-small">📞</a>
          <button class="write-btn-small" onclick="openChat('Мария К.')">✉️</button>
          <button class="respond-btn" onclick="respondToRequest('Мария К.', 'Новосибирск → Славгород')">Откликнуться</button>
        </div>
      </div>
    </div>
    <div class="request-card">
      <div class="request-main">
        <div class="request-route"><div class="request-route-text"><div class="request-from">Барнаул</div><div style="display: flex; align-items: center; gap: 6px; margin: 4px 0;"><svg class="request-arrow-icon" width="16" height="16" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2"><path d="M5 10h10M10 5l5 5-5 5"/></svg></div><div class="request-to">Горно-Алтайск</div></div></div>
        <div class="request-meta"><div class="request-tag">📅 20 ноября</div><div class="request-tag">👥 2 места</div><div class="request-tag">⏰ День</div></div>
        <div class="request-comment">«Едем с ребёнком, нужно детское кресло. Готовы оплатить заранее.»</div>
      </div>
      <div class="request-footer">
        <div class="request-author"><div class="request-author-avatar" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);">Е</div><div><div class="request-author-name">Елена П.</div><div class="request-author-date">5 часов назад</div><a href="tel:+79993456789" style="font-size: 0.8125rem; color: var(--green); font-weight: 600; text-decoration: none;">📞 +7 (999) 345-67-89</a></div></div>
        <div class="request-actions">
          <a href="tel:+79993456789" class="call-btn-small">📞</a>
          <button class="write-btn-small" onclick="openChat('Елена П.')">✉️</button>
          <button class="respond-btn" onclick="respondToRequest('Елена П.', 'Барнаул → Горно-Алтайск')">Откликнуться</button>
        </div>
      </div>
    </div>
    <div class="request-card">
      <div class="request-main">
        <div class="request-route"><div class="request-route-text"><div class="request-from">Славгород</div><div style="display: flex; align-items: center; gap: 6px; margin: 4px 0;"><svg class="request-arrow-icon" width="16" height="16" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2"><path d="M5 10h10M10 5l5 5-5 5"/></svg></div><div class="request-to">Новосибирск</div></div></div>
        <div class="request-meta"><div class="request-tag">📅 21 ноября</div><div class="request-tag">👥 1 место</div><div class="request-tag">⏰ Вечер</div></div>
        <div class="request-comment">«Можно в любое время после 17:00. Не к спеху.»</div>
      </div>
      <div class="request-footer">
        <div class="request-author"><div class="request-author-avatar" style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);">Д</div><div><div class="request-author-name">Дмитрий В.</div><div class="request-author-date">Вчера</div><a href="tel:+79994567890" style="font-size: 0.8125rem; color: var(--green); font-weight: 600; text-decoration: none;">📞 +7 (999) 456-78-90</a></div></div>
        <div class="request-actions">
          <a href="tel:+79994567890" class="call-btn-small">📞</a>
          <button class="write-btn-small" onclick="openChat('Дмитрий В.')">✉️</button>
          <button class="respond-btn" onclick="respondToRequest('Дмитрий В.', 'Славгород → Новосибирск')">Откликнуться</button>
        </div>
      </div>
    </div>
    <div class="request-card">
      <div class="request-main">
        <div class="request-route"><div class="request-route-text"><div class="request-from">Бийск</div><div style="display: flex; align-items: center; gap: 6px; margin: 4px 0;"><svg class="request-arrow-icon" width="16" height="16" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2"><path d="M5 10h10M10 5l5 5-5 5"/></svg></div><div class="request-to">Барнаул</div></div></div>
        <div class="request-meta"><div class="request-tag">📅 22 ноября</div><div class="request-tag">👥 3 места</div><div class="request-tag">⏰ Утро</div></div>
        <div class="request-comment">«Нас трое, едем на мероприятие. Нужен вместительный автомобиль.»</div>
      </div>
      <div class="request-footer">
        <div class="request-author"><div class="request-author-avatar" style="background: linear-gradient(135deg, #fa709a 0%, #fee140 100%);">А</div><div><div class="request-author-name">Анна С.</div><div class="request-author-date">2 дня назад</div><a href="tel:+79995678901" style="font-size: 0.8125rem; color: var(--green); font-weight: 600; text-decoration: none;">📞 +7 (999) 567-89-01</a></div></div>
        <div class="request-actions">
          <a href="tel:+79995678901" class="call-btn-small">📞</a>
          <button class="write-btn-small" onclick="openChat('Анна С.')">✉️</button>
          <button class="respond-btn" onclick="respondToRequest('Анна С.', 'Бийск → Барнаул')">Откликнуться</button>
        </div>
      </div>
    </div>
  </div>
  <div class="bottom-nav">
    <div class="nav-item" onclick="showScreen('screen-home')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="7"/><path d="M21 21l-4.35-4.35"/></svg><span class="nav-label">Искать</span></div>
    <div class="nav-item" onclick="showScreen('screen-publish')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="9"/><path d="M12 8v8M8 12h8"/></svg><span class="nav-label">Опубликовать</span></div>
    <div class="nav-item active" onclick="showScreen('screen-requests')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 00-3-3.87M16 3.13a4 4 0 010 7.75"/></svg><span class="nav-label">Заявки</span></div>
    <div class="nav-item" onclick="showScreen('screen-trips')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 17H5a2 2 0 01-2-2V5a2 2 0 012-2h4M15 7h4a2 2 0 012 2v10a2 2 0 01-2 2h-4"/><circle cx="9" cy="17" r="2"/><circle cx="15" cy="7" r="2"/></svg><span class="nav-label">Поездки</span></div>
    <div class="nav-item" onclick="showScreen('screen-profile')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/><circle cx="12" cy="7" r="4"/></svg><span class="nav-label">Профиль</span></div>
  </div>
</div>

<!-- TRIPS -->
<div class="screen" id="screen-trips">
  <div class="header">
    <div class="header-back" onclick="showScreen('screen-home')"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#00B87C" stroke-width="2.5" stroke-linecap="round"><path d="M19 12H5M12 19l-7-7 7-7"/></svg></div>
    <div class="header-title">Ваши поездки</div>
  </div>
  <div class="tabs" style="margin-bottom: 16px;">
    <div class="tab active"><div class="tab-label">Предстоящие</div></div>
    <div class="tab"><div class="tab-label">Прошедшие</div></div>
  </div>
  <div class="trip-card" onclick="showScreen('screen-detail')">
    <div class="trip-main">
      <div class="trip-times"><div class="trip-departure">17:00</div><div class="trip-duration">5 ч 30</div><div class="trip-arrival">22:30</div></div>
      <div class="trip-route-line"><div class="trip-route-dot"></div><div class="trip-route-bar"></div><div class="trip-route-dot arrival"></div></div>
      <div class="trip-cities"><div class="trip-city">Новосибирск</div><div class="trip-city">Славгород</div></div>
      <div class="trip-price"><div class="trip-price-value">1 850<sup>,00 ₽</sup></div></div>
    </div>
    <div class="trip-driver">
      <div class="driver-avatar verified">В</div>
      <div class="driver-info"><div class="driver-name">Василий</div><div class="driver-rating"><span class="star">★</span> 5</div><a href="tel:+79991234567" style="font-size: 0.875rem; color: var(--green); font-weight: 600; text-decoration: none;">📞 +7 (999) 123-45-67</a></div>
      <div class="action-buttons">
        <a href="tel:+79991234567" class="call-btn-small">📞</a>
        <button class="write-btn-small" onclick="openChat('Василий')">✉️</button>
        <div style="background: var(--green-light); color: var(--green); padding: 6px 12px; border-radius: 20px; font-size: 0.75rem; font-weight: 700;">Ожидает</div>
      </div>
    </div>
  </div>
  <div class="bottom-nav">
    <div class="nav-item" onclick="showScreen('screen-home')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="7"/><path d="M21 21l-4.35-4.35"/></svg><span class="nav-label">Искать</span></div>
    <div class="nav-item" onclick="showScreen('screen-publish')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="9"/><path d="M12 8v8M8 12h8"/></svg><span class="nav-label">Опубликовать</span></div>
    <div class="nav-item" onclick="showScreen('screen-requests')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 00-3-3.87M16 3.13a4 4 0 010 7.75"/></svg><span class="nav-label">Заявки</span></div>
    <div class="nav-item active" onclick="showScreen('screen-trips')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 17H5a2 2 0 01-2-2V5a2 2 0 012-2h4M15 7h4a2 2 0 012 2v10a2 2 0 01-2 2h-4"/><circle cx="9" cy="17" r="2"/><circle cx="15" cy="7" r="2"/></svg><span class="nav-label">Поездки</span></div>
    <div class="nav-item" onclick="showScreen('screen-profile')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/><circle cx="12" cy="7" r="4"/></svg><span class="nav-label">Профиль</span></div>
  </div>
</div>

<!-- PROFILE -->
<div class="screen" id="screen-profile">
  <div class="header">
    <div class="header-back" onclick="showScreen('screen-home')"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#00B87C" stroke-width="2.5" stroke-linecap="round"><path d="M19 12H5M12 19l-7-7 7-7"/></svg></div>
    <div class="header-title">Профиль</div>
  </div>
  <div class="profile-header">
    <div class="profile-avatar">А</div>
    <div class="profile-name">Андрей</div>
    <div class="profile-rating"><span style="color: #FFB800;">★</span> 4.9 - 28 отзывов</div>
  </div>
  <div class="profile-stats">
    <div class="profile-stat"><div class="profile-stat-value">12</div><div class="profile-stat-label">Поездок</div></div>
    <div class="profile-stat"><div class="profile-stat-value">4.9</div><div class="profile-stat-label">Рейтинг</div></div>
    <div class="profile-stat"><div class="profile-stat-value">28</div><div class="profile-stat-label">Отзывов</div></div>
  </div>
  <div class="profile-menu">
    <div class="profile-menu-item" onclick="showToast('Редактирование профиля')">
      <div class="profile-menu-icon"><svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 3l3 3-9 9H5v-3z"/></svg></div>
      <div class="profile-menu-text">Редактировать профиль</div>
      <div class="profile-menu-arrow">›</div>
    </div>
    <div class="profile-menu-item" onclick="togglePhoneVisibility(this)">
      <div class="profile-menu-icon"><svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/></svg></div>
      <div class="profile-menu-text" style="flex: 1;">Показывать номер телефона</div>
      <div class="access-toggle active" id="phoneVisibilityToggle" style="margin-left: 0;"></div>
    </div>
    <div class="profile-menu-item" onclick="showToast('Мои документы')">
      <div class="profile-menu-icon"><svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 3h8l4 4v10a1 1 0 01-1 1H4a1 1 0 01-1-1V4a1 1 0 011-1z"/><path d="M12 3v4h4"/></svg></div>
      <div class="profile-menu-text">Мои документы</div><div class="profile-menu-arrow">›</div>
    </div>
    <div class="profile-menu-item" onclick="showToast('Способы оплаты')">
      <div class="profile-menu-icon"><svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="5" width="16" height="11" rx="2"/><path d="M2 9h16"/></svg></div>
      <div class="profile-menu-text">Способы оплаты</div><div class="profile-menu-arrow">›</div>
    </div>
    <div class="profile-menu-item" onclick="showToast('Настройки')">
      <div class="profile-menu-icon"><svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2"><circle cx="10" cy="10" r="3"/><path d="M10 2v2M10 16v2M2 10h2M16 10h2"/></svg></div>
      <div class="profile-menu-text">Настройки</div><div class="profile-menu-arrow">›</div>
    </div>
    <div class="profile-menu-item" onclick="showToast('Помощь и поддержка')">
      <div class="profile-menu-icon"><svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2"><circle cx="10" cy="10" r="8"/><path d="M7.5 7.5a2.5 2.5 0 015 0c0 1.5-2.5 2-2.5 3.5M10 14v.01"/></svg></div>
      <div class="profile-menu-text">Помощь и поддержка</div><div class="profile-menu-arrow">›</div>
    </div>
  </div>
  <div style="height: 20px;"></div>
  <div class="bottom-nav">
    <div class="nav-item" onclick="showScreen('screen-home')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="7"/><path d="M21 21l-4.35-4.35"/></svg><span class="nav-label">Искать</span></div>
    <div class="nav-item" onclick="showScreen('screen-publish')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="9"/><path d="M12 8v8M8 12h8"/></svg><span class="nav-label">Опубликовать</span></div>
    <div class="nav-item" onclick="showScreen('screen-requests')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 00-3-3.87M16 3.13a4 4 0 010 7.75"/></svg><span class="nav-label">Заявки</span></div>
    <div class="nav-item" onclick="showScreen('screen-trips')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 17H5a2 2 0 01-2-2V5a2 2 0 012-2h4M15 7h4a2 2 0 012 2v10a2 2 0 01-2 2h-4"/><circle cx="9" cy="17" r="2"/><circle cx="15" cy="7" r="2"/></svg><span class="nav-label">Поездки</span></div>
    <div class="nav-item active" onclick="showScreen('screen-profile')"><svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/><circle cx="12" cy="7" r="4"/></svg><span class="nav-label">Профиль</span></div>
  </div>
</div>

<!-- Passenger Modal -->
<div class="map-modal-overlay" id="passengerModal">
  <div class="map-modal-content" style="height: auto; max-height: 50vh;">
    <div class="map-modal-header">
      <div class="map-modal-title">Количество пассажиров</div>
      <button class="map-modal-close" onclick="document.getElementById('passengerModal').classList.remove('active')">✕</button>
    </div>
    <div style="padding: 20px; display: flex; align-items: center; justify-content: center; gap: 24px;">
      <button onclick="changePassengers(-1)" style="width: 48px; height: 48px; border-radius: 50%; border: 2px solid var(--border); background: white; font-size: 1.5rem; cursor: pointer;">−</button>
      <div style="font-size: 2rem; font-weight: 800; min-width: 60px; text-align: center;" id="passengerCount">1</div>
      <button onclick="changePassengers(1)" style="width: 48px; height: 48px; border-radius: 50%; border: 2px solid var(--border); background: white; font-size: 1.5rem; cursor: pointer;">+</button>
    </div>
    <button class="search-btn" onclick="confirmPassengers()" style="margin: 0 20px 20px;">Готово</button>
  </div>
</div>

<!-- Request Modal -->
<div class="map-modal-overlay" id="requestModal">
  <div class="map-modal-content" style="height: auto; max-height: 85vh;">
    <div class="map-modal-header">
      <div class="map-modal-title">Новая заявка</div>
      <button class="map-modal-close" onclick="document.getElementById('requestModal').classList.remove('active')">✕</button>
    </div>
    <div style="padding: 20px; overflow-y: auto;">
      <div class="type-toggle" style="margin-bottom: 16px;">
        <div class="type-toggle-btn active" id="reqTogglePassenger" onclick="switchRequestType('passenger')">👥 Я пассажир</div>
        <div class="type-toggle-btn" id="reqToggleParcel" onclick="switchRequestType('parcel')">📦 Посылка</div>
      </div>
      <div style="margin-bottom: 14px;"><div style="font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 6px; font-weight: 500;">Откуда</div><input class="form-input-field" id="reqFrom" placeholder="Город отправления" style="width: 100%; padding: 12px; border: 1.5px solid var(--border); border-radius: 12px; font-size: 1rem; font-family: inherit; outline: none;"></div>
      <div style="margin-bottom: 14px;"><div style="font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 6px; font-weight: 500;">Куда</div><input class="form-input-field" id="reqTo" placeholder="Город назначения" style="width: 100%; padding: 12px; border: 1.5px solid var(--border); border-radius: 12px; font-size: 1rem; font-family: inherit; outline: none;"></div>
      <div style="margin-bottom: 14px;"><div style="font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 6px; font-weight: 500;">Дата</div><input type="date" id="reqDate" style="width: 100%; padding: 12px; border: 1.5px solid var(--border); border-radius: 12px; font-size: 1rem; font-family: inherit; outline: none; color: var(--text);"></div>
      <div style="margin-bottom: 14px;"><div style="font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 6px; font-weight: 500;">Время предпочтительно</div><select id="reqTime" style="width: 100%; padding: 12px; border: 1.5px solid var(--border); border-radius: 12px; font-size: 1rem; font-family: inherit; outline: none; background: white; color: var(--text);"><option>Утро (до 12:00)</option><option>День (12:00 - 17:00)</option><option>Вечер (после 17:00)</option><option>Любое время</option></select></div>
      <div id="reqPassengerFields">
        <div style="margin-bottom: 14px;"><div style="font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 6px; font-weight: 500;">Количество мест</div><input type="number" id="reqSeats" value="1" min="1" max="6" style="width: 100%; padding: 12px; border: 1.5px solid var(--border); border-radius: 12px; font-size: 1rem; font-family: inherit; outline: none;"></div>
      </div>
      <div id="reqParcelFields" class="hidden">
        <div style="margin-bottom: 14px;"><div style="font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 6px; font-weight: 500;">Тип груза</div>
          <select id="reqCargoType" onchange="toggleCustomCargo()" style="width: 100%; padding: 12px; border: 1.5px solid var(--border); border-radius: 12px; font-size: 1rem; font-family: inherit; outline: none; background: white; color: var(--text);">
            <option>Документы</option><option>Коробка</option><option>Хрупкое</option><option>Одежда</option><option>Электроника</option><option>Продукты</option><option value="custom">Другое (указать вручную)</option>
          </select>
        </div>
        <div style="margin-bottom: 14px;" id="customCargoField" class="hidden">
          <div style="font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 6px; font-weight: 500;">Укажите тип груза</div>
          <input type="text" id="reqCustomCargo" placeholder="Например: одежда, электроника, продукты..." style="width: 100%; padding: 12px; border: 1.5px solid var(--border); border-radius: 12px; font-size: 1rem; font-family: inherit; outline: none;">
        </div>
        <div style="margin-bottom: 14px;"><div style="font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 6px; font-weight: 500;">Описание груза</div><textarea id="reqCargoDesc" placeholder="Что отправляете?" style="width: 100%; padding: 12px; border: 1.5px solid var(--border); border-radius: 12px; font-size: 1rem; font-family: inherit; outline: none; resize: vertical; min-height: 60px;"></textarea></div>
        <div style="margin-bottom: 14px;"><div style="font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 6px; font-weight: 500;">Объявленная ценность (₽)</div><input type="number" id="reqCargoValue" placeholder="Например: 5000" min="0" style="width: 100%; padding: 12px; border: 1.5px solid var(--border); border-radius: 12px; font-size: 1rem; font-family: inherit; outline: none;"></div>
        <div style="margin-bottom: 14px;"><div style="font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 6px; font-weight: 500;">Кому передать (ФИО, телефон)</div><input type="text" id="reqRecipient" placeholder="Иванов И.И., +7 (999) 123-45-67" style="width: 100%; padding: 12px; border: 1.5px solid var(--border); border-radius: 12px; font-size: 1rem; font-family: inherit; outline: none;"></div>
      </div>
      <div style="margin-bottom: 14px;"><div style="font-size: 0.8125rem; color: var(--text-secondary); margin-bottom: 6px; font-weight: 500;">Комментарий (необязательно)</div><textarea id="reqComment" placeholder="Багаж, особенности, пожелания..." style="width: 100%; padding: 12px; border: 1.5px solid var(--border); border-radius: 12px; font-size: 1rem; font-family: inherit; outline: none; resize: vertical; min-height: 80px;"></textarea></div>
      <button class="search-btn" onclick="submitRequest()">Опубликовать заявку</button>
    </div>
  </div>
</div>

<!-- Map Modal -->
<div class="map-modal-overlay" id="mapModal">
  <div class="map-modal-content">
    <div class="map-modal-header">
      <div class="map-modal-title" id="mapModalTitle">Место на карте</div>
      <button class="map-modal-close" onclick="closeMap()">✕</button>
    </div>
    <div class="map-container"><div id="map"></div></div>
    <div class="map-info-bar" id="mapInfoBar">Загрузка карты...</div>
  </div>
</div>

<!-- Accessibility Modal -->
<div class="access-modal-overlay" id="accessModal">
  <div class="access-modal-content">
    <div class="access-modal-handle"></div>
    <div class="access-modal-title">♿ Настройки доступности</div>
    <div class="access-option">
      <div class="access-option-info">
        <div class="access-option-title">🔤 Размер текста</div>
        <div class="access-option-desc">Увеличьте шрифт для удобства чтения</div>
        <div class="access-font-sizes">
          <button class="access-font-btn active" onclick="setFontSize(16, this)" style="font-size: 14px;">А</button>
          <button class="access-font-btn" onclick="setFontSize(20, this)" style="font-size: 17px;">А</button>
          <button class="access-font-btn" onclick="setFontSize(24, this)" style="font-size: 20px;">А</button>
        </div>
      </div>
    </div>
    <div class="access-option">
      <div class="access-option-info"><div class="access-option-title">🌙 Высокая контрастность</div><div class="access-option-desc">Чёрный фон, белый текст</div></div>
      <div class="access-toggle" id="contrastToggle" onclick="toggleContrast()"></div>
    </div>
    <div class="access-option">
      <div class="access-option-info"><div class="access-option-title">🔊 Озвучка сообщений</div><div class="access-option-desc">ИИ-ассистент читает ответы вслух</div></div>
      <div class="access-toggle active" id="voiceToggle" onclick="toggleVoice()"></div>
    </div>
    <div class="access-option">
      <div class="access-option-info"><div class="access-option-title">📝 Субтитры для глухих</div><div class="access-option-desc">Всегда показывать текст ответов</div></div>
      <div class="access-toggle active" id="subtitleToggle" onclick="toggleSubtitle()"></div>
    </div>
    <div class="access-option">
      <div class="access-option-info"><div class="access-option-title">👆 Увеличенные кнопки</div><div class="access-option-desc">Для людей с нарушением моторики</div></div>
      <div class="access-toggle" id="bigButtonsToggle" onclick="toggleBigButtons()"></div>
    </div>
    <button class="search-btn" onclick="closeAccessModal()" style="margin-top: 10px;">Закрыть</button>
  </div>
</div>

<!-- AI Modal - Voice Assistant -->
<div class="ai-modal-overlay" id="aiModal">
  <div class="ai-modal-content">
    <div class="ai-header">
      <div class="ai-header-avatar">🤖</div>
      <div class="ai-header-info"><div class="ai-header-name">Помощник Попутчик</div><div class="ai-header-status">Онлайн • Голосовой режим</div></div>
      <button class="ai-close" onclick="closeAiModal()">✕</button>
    </div>

    <!-- Speed control -->
    <div class="speed-control">
      <label>🗣️ Скорость речи:</label>
      <select id="speechSpeed" onchange="updateSpeechSpeed()">
        <option value="0.7">Медленно (для пожилых)</option>
        <option value="0.9" selected>Нормально</option>
        <option value="1.1">Быстро</option>
      </select>
    </div>

    <div class="ai-messages" id="aiMessages">
      <div class="ai-message bot">Здравствуйте, Андрей! 👋 Я ваш голосовой помощник. Нажмите на микрофон и скажите, что вам нужно. Например: "Найди поездку завтра в Кемерово" или "Оставь заявку на 2 места".<span class="msg-time">сейчас</span></div>
    </div>

    <!-- Voice visualizer -->
    <div class="voice-visualizer" id="voiceVisualizer">
      <div class="voice-bar"></div><div class="voice-bar"></div><div class="voice-bar"></div><div class="voice-bar"></div><div class="voice-bar"></div><div class="voice-bar"></div><div class="voice-bar"></div>
    </div>
    <div class="voice-status" id="voiceStatus">Нажмите на микрофон, чтобы начать</div>

    <div class="ai-quick-replies" id="aiQuickReplies">
      <button class="ai-quick-reply" onclick="sendQuickReply('Найти поездку завтра')">🔍 Найти поездку</button>
      <button class="ai-quick-reply" onclick="sendQuickReply('Оставь заявку на 2 места')">📝 Оставить заявку</button>
      <button class="ai-quick-reply" onclick="sendQuickReply('Сколько стоит поездка?')">💰 Узнать цены</button>
      <button class="ai-quick-reply" onclick="sendQuickReply('Помощь')">❓ Помощь</button>
    </div>
    <div class="ai-input-area">
      <button class="ai-voice-btn" id="voiceBtn" onclick="toggleVoiceInput()" aria-label="Голосовой ввод">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 1a3 3 0 00-3 3v8a3 3 0 006 0V4a3 3 0 00-3-3z"/><path d="M19 10v2a7 7 0 01-14 0v-2M12 19v4M8 23h8"/></svg>
      </button>
      <input type="text" class="ai-input" id="aiInput" placeholder="Или напишите сообщение..." onkeypress="if(event.key==='Enter')sendAiMessage()">
      <button class="ai-send-btn" onclick="sendAiMessage()">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M22 2L11 13M22 2l-7 20-4-9-9-4 20-7z"/></svg>
      </button>
    </div>
  </div>
</div>

<!-- AI FAB -->
<button class="ai-fab" onclick="openAiModal()" aria-label="ИИ-помощник">
  <svg width="32" height="32" viewBox="0 0 32 32" fill="none">
    <circle cx="16" cy="16" r="14" fill="white" opacity="0.2"/>
    <path d="M10 14c0-3.3 2.7-6 6-6s6 2.7 6 6" stroke="white" stroke-width="2.5" stroke-linecap="round"/>
    <circle cx="12" cy="13" r="1.5" fill="white"/><circle cx="20" cy="13" r="1.5" fill="white"/>
    <path d="M12 18c1 1.5 2.5 2.5 4 2.5s3-1 4-2.5" stroke="white" stroke-width="2" stroke-linecap="round"/>
    <circle cx="24" cy="8" r="3" fill="#FFD700"/>
    <path d="M23 8l1 1 2-2" stroke="white" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
  </svg>
</button>

<script>
(function() {
  'use strict';

  var state = { from: '', to: '', date: 'Сегодня', passengers: 1, searchType: 'passengers', publishType: 'passengers', requestType: 'passenger', phoneVisibility: true };
  var accessSettings = { fontSize: 16, contrast: false, voice: true, subtitle: true, bigButtons: false, speechSpeed: 0.9 };
  var mapInstance = null;
  var siberiaMapInstance = null;
  var recognition = null;
  var subtitleTimeout = null;
  var speechUnlocked = false;
  var isListening = false;

  // Multi-turn dialog state
  var dialogState = {
    active: false,
    intent: null,
    step: 0,
    data: {}
  };

  var cities = [
    { name: 'Новосибирск', region: 'Новосибирская обл.', lat: 55.0084, lng: 82.9357 },
    { name: 'Славгород', region: 'Алтайский край', lat: 52.9928, lng: 78.6639 },
    { name: 'Барнаул', region: 'Алтайский край', lat: 53.3606, lng: 83.7636 },
    { name: 'Бийск', region: 'Алтайский край', lat: 52.5378, lng: 85.2083 },
    { name: 'Рубцовск', region: 'Алтайский край', lat: 51.5208, lng: 81.2089 },
    { name: 'Новоалтайск', region: 'Алтайский край', lat: 53.4108, lng: 83.9339 },
    { name: 'Заринск', region: 'Алтайский край', lat: 53.7078, lng: 84.9389 },
    { name: 'Камень-на-Оби', region: 'Алтайский край', lat: 53.7928, lng: 81.3539 },
    { name: 'Алейск', region: 'Алтайский край', lat: 52.5028, lng: 82.7839 },
    { name: 'Яровое', region: 'Алтайский край', lat: 52.9228, lng: 78.5739 },
    { name: 'Кулунда', region: 'Алтайский край', lat: 52.2928, lng: 78.8639 },
    { name: 'Горно-Алтайск', region: 'Республика Алтай', lat: 51.9588, lng: 85.9603 },
    { name: 'Майма', region: 'Республика Алтай', lat: 52.0928, lng: 85.8139 },
    { name: 'Кемерово', region: 'Кемеровская обл.', lat: 55.3333, lng: 86.0833 },
    { name: 'Новокузнецк', region: 'Кемеровская обл.', lat: 53.7557, lng: 87.1099 },
    { name: 'Прокопьевск', region: 'Кемеровская обл.', lat: 53.9061, lng: 86.7194 },
    { name: 'Ленинск-Кузнецкий', region: 'Кемеровская обл.', lat: 54.6667, lng: 86.1833 },
    { name: 'Междуреченск', region: 'Кемеровская обл.', lat: 53.6833, lng: 88.0667 },
    { name: 'Киселёвск', region: 'Кемеровская обл.', lat: 54.0000, lng: 86.6333 },
    { name: 'Белово', region: 'Кемеровская обл.', lat: 54.4167, lng: 86.3000 },
    { name: 'Омск', region: 'Омская обл.', lat: 54.9885, lng: 73.3242 },
    { name: 'Тара', region: 'Омская обл.', lat: 56.9000, lng: 74.3667 },
    { name: 'Исилькуль', region: 'Омская обл.', lat: 54.9167, lng: 71.2833 },
    { name: 'Калачинск', region: 'Омская обл.', lat: 55.0667, lng: 74.5667 },
    { name: 'Тюкалинск', region: 'Омская обл.', lat: 55.8667, lng: 72.2000 },
    { name: 'Называевск', region: 'Омская обл.', lat: 55.5667, lng: 71.3500 },
    { name: 'Томск', region: 'Томская обл.', lat: 56.5018, lng: 84.9744 },
    { name: 'Красноярск', region: 'Красноярский край', lat: 56.0153, lng: 92.8932 },
    { name: 'Москва', region: 'Московская обл.', lat: 55.7558, lng: 37.6173 },
    { name: 'Санкт-Петербург', region: 'Ленинградская обл.', lat: 59.9311, lng: 30.3609 },
    { name: 'Казань', region: 'Татарстан', lat: 55.8304, lng: 49.0661 },
    { name: 'Тула', region: 'Тульская обл.', lat: 54.2044, lng: 37.6155 }
  ];

  var busStations = [
    { city: 'Новосибирск', region: 'Новосибирская обл.', address: 'ул. Щетинкина, 68', phone: '+7 (383) 222-22-22', phoneAlt: '+7 (383) 222-33-33', hours: 'Круглосуточно', lat: 55.0301, lng: 82.9211 },
    { city: 'Кемерово', region: 'Кемеровская обл.', address: 'пр. Советский, 63', phone: '+7 (3842) 36-66-66', phoneAlt: '+7 (3842) 36-77-77', hours: '05:00 – 23:00', lat: 55.3333, lng: 86.0833 },
    { city: 'Новокузнецк', region: 'Кемеровская обл.', address: 'ул. Кирова, 33', phone: '+7 (3843) 74-55-55', phoneAlt: '+7 (3843) 74-66-66', hours: '05:30 – 22:00', lat: 53.7557, lng: 87.1099 },
    { city: 'Омск', region: 'Омская обл.', address: 'ул. Станция Омск-Пасс., 1', phone: '+7 (3812) 65-55-55', phoneAlt: '+7 (3812) 65-66-66', hours: 'Круглосуточно', lat: 54.9885, lng: 73.3242 },
    { city: 'Барнаул', region: 'Алтайский край', address: 'Площадь Сахарова, 2', phone: '+7 (3852) 63-01-01', phoneAlt: '+7 (3852) 63-02-02', hours: 'Круглосуточно', lat: 53.3606, lng: 83.7636 },
    { city: 'Бийск', region: 'Алтайский край', address: 'ул. Льва Толстого, 37', phone: '+7 (3854) 33-22-33', phoneAlt: '+7 (3854) 33-44-55', hours: '05:00 – 22:00', lat: 52.5378, lng: 85.2083 },
    { city: 'Рубцовск', region: 'Алтайский край', address: 'ул. Тракторная, 1', phone: '+7 (38557) 4-22-33', phoneAlt: '+7 (38557) 4-15-67', hours: '05:30 – 21:00', lat: 51.5208, lng: 81.2089 },
    { city: 'Славгород', region: 'Алтайский край', address: 'ул. Ленина, 81', phone: '+7 (38552) 2-12-34', phoneAlt: '+7 (38552) 2-15-78', hours: '06:00 – 21:00', lat: 52.9928, lng: 78.6639 },
    { city: 'Алейск', region: 'Алтайский край', address: 'ул. Советская, 45', phone: '+7 (38553) 2-15-67', phoneAlt: null, hours: '06:00 – 20:00', lat: 52.5028, lng: 82.7839 },
    { city: 'Новоалтайск', region: 'Алтайский край', address: 'ул. Пархоменко, 20', phone: '+7 (38532) 2-23-45', phoneAlt: null, hours: '05:30 – 21:30', lat: 53.4108, lng: 83.9339 },
    { city: 'Заринск', region: 'Алтайский край', address: 'ул. Гагарина, 12', phone: '+7 (38555) 2-12-34', phoneAlt: null, hours: '06:00 – 20:30', lat: 53.7078, lng: 84.9389 },
    { city: 'Камень-на-Оби', region: 'Алтайский край', address: 'ул. Ленина, 56', phone: '+7 (38584) 2-12-34', phoneAlt: null, hours: '06:00 – 20:00', lat: 53.7928, lng: 81.3539 },
    { city: 'Яровое', region: 'Алтайский край', address: 'ул. Советская, 18', phone: '+7 (38552) 6-12-34', phoneAlt: null, hours: '07:00 – 20:00', lat: 52.9228, lng: 78.5739 },
    { city: 'Кулунда', region: 'Алтайский край', address: 'ул. Привокзальная, 3', phone: '+7 (38552) 5-12-34', phoneAlt: null, hours: '06:30 – 20:00', lat: 52.2928, lng: 78.8639 },
    { city: 'Горно-Алтайск', region: 'Республика Алтай', address: 'Коммунистический пр., 77', phone: '+7 (38822) 6-12-34', phoneAlt: '+7 (38822) 6-45-67', hours: '06:00 – 21:00', lat: 51.9588, lng: 85.9603 },
    { city: 'Майма', region: 'Республика Алтай', address: 'ул. Ленина, 34', phone: '+7 (38822) 5-23-45', phoneAlt: null, hours: '07:00 – 19:00', lat: 52.0928, lng: 85.8139 },
    { city: 'Томск', region: 'Томская обл.', address: 'ул. Привокзальная, 1', phone: '+7 (3822) 65-00-00', phoneAlt: '+7 (3822) 65-11-11', hours: '05:00 – 23:00', lat: 56.5018, lng: 84.9744 },
    { city: 'Красноярск', region: 'Красноярский край', address: 'пр. им. Газеты Красноярский Рабочий, 100', phone: '+7 (391) 228-77-77', phoneAlt: null, hours: 'Круглосуточно', lat: 56.0153, lng: 92.8932 },
    { city: 'Ленинск-Кузнецкий', region: 'Кемеровская обл.', address: 'ул. Кирова, 15', phone: '+7 (38456) 2-22-22', phoneAlt: null, hours: '06:00 – 21:00', lat: 54.6667, lng: 86.1833 },
    { city: 'Прокопьевск', region: 'Кемеровская обл.', address: 'ул. Транспортная, 8', phone: '+7 (3846) 62-33-33', phoneAlt: null, hours: '06:00 – 21:00', lat: 53.9061, lng: 86.7194 }
  ];

  var availableRides = [
    { driver: 'Василий', phone: '+7 (999) 123-45-67', phoneClean: '+79991234567', from: 'Новосибирск', to: 'Славгород', date: '19 ноября', time: '17:00', price: '1 850 ₽', rating: '5/5 (102 отзыва)' },
    { driver: 'Сергей', phone: '+7 (999) 111-22-33', phoneClean: '+79991112233', from: 'Новосибирск', to: 'Славгород', date: '19 ноября', time: '07:00', price: '1 700 ₽', rating: '4.8 (56 отзывов)', seats: 'Мест нет' },
    { driver: 'Алексей', phone: '+7 (999) 222-33-44', phoneClean: '+79992223344', from: 'Кемерово', to: 'Новосибирск', date: '19 ноября', time: '14:00', price: '1 200 ₽', rating: '4.9 (78 отзывов)' },
    { driver: 'Ирина', phone: '+7 (999) 333-44-55', phoneClean: '+79993334455', from: 'Омск', to: 'Новосибирск', date: '20 ноября', time: '09:00', price: '1 400 ₽', rating: '5/5 (34 отзыва)' },
    { driver: 'Дмитрий', phone: '+7 (999) 444-55-66', phoneClean: '+79994445566', from: 'Барнаул', to: 'Горно-Алтайск', date: '20 ноября', time: '11:00', price: '900 ₽', rating: '4.7 (45 отзывов)' }
  ];

  window.showScreen = function(id) {
    var screens = document.querySelectorAll('.screen');
    for (var i = 0; i < screens.length; i++) screens[i].classList.remove('active');
    var el = document.getElementById(id);
    if (el) el.classList.add('active');
    window.scrollTo(0, 0);
    closeAllAutocomplete();
    if (id === 'screen-home') {
      setTimeout(initSiberiaMap, 400);
    }
  };

  window.showToast = function(msg) {
    var t = document.getElementById('toast');
    t.textContent = msg;
    t.classList.add('show');
    setTimeout(function() { t.classList.remove('show'); }, 2500);
  };

  window.showPassengerModal = function() { document.getElementById('passengerModal').classList.add('active'); };
  window.changePassengers = function(delta) {
    state.passengers = Math.max(1, Math.min(8, state.passengers + delta));
    document.getElementById('passengerCount').textContent = state.passengers;
  };
  window.confirmPassengers = function() {
    var word = state.passengers === 1 ? 'пассажир' : state.passengers < 5 ? 'пассажира' : 'пассажиров';
    document.getElementById('homePassengers').textContent = state.passengers + ' ' + word;
    document.getElementById('passengerModal').classList.remove('active');
  };

  window.switchSearchType = function(type) {
    state.searchType = type;
    document.getElementById('togglePassengers').classList.toggle('active', type === 'passengers');
    document.getElementById('toggleParcels').classList.toggle('active', type === 'parcels');
    if (type === 'passengers') {
      document.getElementById('fromLabel').textContent = 'Откуда';
      document.getElementById('toLabel').textContent = 'Куда';
      document.getElementById('homeFromInput').placeholder = 'Город, вокзал, заведение';
      document.getElementById('homeToInput').placeholder = 'Город, вокзал, заведение';
      document.getElementById('passengersField').classList.remove('hidden');
      document.getElementById('cargoField').classList.add('hidden');
      document.getElementById('customCargoField').classList.add('hidden');
    } else {
      document.getElementById('fromLabel').textContent = 'Город отправления';
      document.getElementById('toLabel').textContent = 'Город назначения';
      document.getElementById('homeFromInput').placeholder = 'Город отправления';
      document.getElementById('homeToInput').placeholder = 'Город назначения';
      document.getElementById('passengersField').classList.add('hidden');
      document.getElementById('cargoField').classList.remove('hidden');
    }
  };

  window.updateCargoDisplay = function() {
    var select = document.getElementById('homeCargoSelect');
    var customField = document.getElementById('customCargoField');
    if (select.value === 'custom') {
      customField.classList.remove('hidden');
    } else {
      customField.classList.add('hidden');
    }
  };

  window.switchPublishType = function(type) {
    state.publishType = type;
    document.getElementById('pubTogglePassengers').classList.toggle('active', type === 'passengers');
    document.getElementById('pubToggleParcels').classList.toggle('active', type === 'parcels');
    if (type === 'passengers') {
      document.getElementById('pubPassengerFields').classList.remove('hidden');
      document.getElementById('pubParcelFields').classList.add('hidden');
    } else {
      document.getElementById('pubPassengerFields').classList.add('hidden');
      document.getElementById('pubParcelFields').classList.remove('hidden');
    }
  };

  window.togglePubCustomCargo = function() {
    var sel = document.getElementById('pubCargoType');
    var customField = document.getElementById('pubCustomCargoField');
    if (sel.value === 'custom') {
      customField.classList.remove('hidden');
    } else {
      customField.classList.add('hidden');
    }
  };

  window.switchRequestType = function(type) {
    state.requestType = type;
    document.getElementById('reqTogglePassenger').classList.toggle('active', type === 'passenger');
    document.getElementById('reqToggleParcel').classList.toggle('active', type === 'parcel');
    if (type === 'passenger') {
      document.getElementById('reqPassengerFields').classList.remove('hidden');
      document.getElementById('reqParcelFields').classList.add('hidden');
    } else {
      document.getElementById('reqPassengerFields').classList.add('hidden');
      document.getElementById('reqParcelFields').classList.remove('hidden');
    }
  };

  window.toggleCustomCargo = function() {
    var sel = document.getElementById('reqCargoType');
    var customField = document.getElementById('customCargoField');
    if (sel.value === 'custom') {
      customField.classList.remove('hidden');
    } else {
      customField.classList.add('hidden');
    }
  };

  function setupAutocomplete(inputId, listId) {
    var input = document.getElementById(inputId);
    var list = document.getElementById(listId);
    if (!input || !list) return;
    input.addEventListener('input', function() {
      var query = this.value.toLowerCase().trim();
      if (query.length < 2) { list.classList.remove('show'); return; }
      var matches = [];
      for (var i = 0; i < cities.length; i++) {
        if (cities[i].name.toLowerCase().indexOf(query) !== -1 || cities[i].region.toLowerCase().indexOf(query) !== -1) {
          matches.push(cities[i]);
          if (matches.length >= 6) break;
        }
      }
      if (matches.length === 0) { list.classList.remove('show'); return; }
      var html = '';
      for (var j = 0; j < matches.length; j++) {
        var h = matches[j].name.replace(new RegExp('(' + query + ')', 'gi'), '<strong>$1</strong>');
        html += '<div class="autocomplete-item" data-name="' + matches[j].name + '">' + h + ', ' + matches[j].region + '</div>';
      }
      list.innerHTML = html;
      list.classList.add('show');
      var items = list.querySelectorAll('.autocomplete-item');
      for (var k = 0; k < items.length; k++) {
        (function(item) {
          item.addEventListener('click', function() {
            input.value = this.getAttribute('data-name');
            list.classList.remove('show');
          });
        })(items[k]);
      }
    });
    input.addEventListener('blur', function() { setTimeout(function() { list.classList.remove('show'); }, 200); });
    input.addEventListener('focus', function() { if (this.value.length >= 2) this.dispatchEvent(new Event('input')); });
  }

  function closeAllAutocomplete() {
    var lists = document.querySelectorAll('.autocomplete-list');
    for (var i = 0; i < lists.length; i++) lists[i].classList.remove('show');
  }

  setupAutocomplete('homeFromInput', 'fromAutocomplete');
  setupAutocomplete('homeToInput', 'toAutocomplete');
  setupAutocomplete('pubFromInput', 'pubFromAutocomplete');
  setupAutocomplete('pubToInput', 'pubToAutocomplete');

  function buildCalendar() {
    var novGrid = document.getElementById('novGrid');
    var decGrid = document.getElementById('decGrid');
    var today = 17;
    var novHTML = '';
    for (var i = 0; i < 6; i++) novHTML += '<div class="calendar-day empty"></div>';
    for (var d = 1; d <= 30; d++) {
      var cls = 'calendar-day';
      if (d < today) cls += ' past';
      else if (d === today) cls += ' available today selected';
      else cls += ' available';
      novHTML += '<div class="' + cls + '" onclick="selectDate(this, ' + d + ', \'ноября\')">' + d + '</div>';
    }
    novGrid.innerHTML = novHTML;
    var decHTML = '<div class="calendar-day empty"></div>';
    for (var dd = 1; dd <= 31; dd++) {
      decHTML += '<div class="calendar-day available" onclick="selectDate(this, ' + dd + ', \'декабря\')">' + dd + '</div>';
    }
    decGrid.innerHTML = decHTML;
  }

  window.selectDate = function(el, day, month) {
    if (el.classList.contains('past')) return;
    var selected = document.querySelectorAll('.calendar-day.selected');
    for (var i = 0; i < selected.length; i++) selected[i].classList.remove('selected');
    el.classList.add('selected');
    var days = ['Вс', 'Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб'];
    var date = new Date(2026, month === 'ноября' ? 10 : 11, day);
    var dayName = days[date.getDay()];
    var shortMonth = month === 'ноября' ? 'нояб.' : 'дек.';
    document.getElementById('homeDate').textContent = dayName + ' ' + day + ' ' + month;
    document.getElementById('pubDate').textContent = day + ' ' + month + ' 2026';
    document.getElementById('resultDate').textContent = dayName + ' ' + day + ' ' + shortMonth + ', ' + state.passengers + ' пассажир';
    setTimeout(function() { showScreen('screen-results'); }, 300);
  };

  window.doSearch = function() {
    var from = document.getElementById('homeFromInput').value;
    var to = document.getElementById('homeToInput').value;
    if (!from || !to) { showToast('Введите города отправления и назначения'); return; }
    document.getElementById('resultRoute').textContent = from + ' → ' + to;
    showScreen('screen-results');
  };

  window.publishTrip = function() {
    var from = document.getElementById('pubFromInput').value;
    var to = document.getElementById('pubToInput').value;
    if (!from || !to) { showToast('Заполните города'); return; }
    if (state.publishType === 'parcels') {
      var cargoType = document.getElementById('pubCargoType').value;
      if (cargoType === 'custom') {
        var customCargo = document.getElementById('pubCustomCargo').value;
        if (!customCargo) { showToast('Укажите тип груза'); return; }
      }
      var weight = document.getElementById('pubWeight').value;
      var price = document.getElementById('pubParcelPrice').value;
      if (!weight || weight <= 0) { showToast('Укажите максимальный вес'); return; }
      if (weight > 15) { showToast('Максимальный вес не может превышать 15 кг'); return; }
      if (!price || price <= 0) { showToast('Укажите стоимость доставки'); return; }
    }
    showToast('Поездка опубликована! ✓');
    setTimeout(function() { showScreen('screen-trips'); }, 1000);
  };

  window.bookTrip = function() { showToast('Запрос на бронирование отправлен!'); };

  function buildDirectory(filter) {
    filter = filter || '';
    var accordion = document.getElementById('directoryAccordion');
    var query = filter.toLowerCase().trim();
    var filtered = [];
    for (var i = 0; i < busStations.length; i++) {
      if (query.length < 2 || busStations[i].city.toLowerCase().indexOf(query) !== -1 || busStations[i].region.toLowerCase().indexOf(query) !== -1) {
        filtered.push(busStations[i]);
      }
    }
    if (filtered.length === 0) { accordion.innerHTML = '<div class="no-results">😔 Город не найден</div>'; return; }
    var html = '';
    for (var j = 0; j < filtered.length; j++) {
      var s = filtered[j];
      var phoneHref = s.phone.replace(/[+\s\(\)]/g, '').replace(/-/g, '');
      html += '<div class="accordion-item">';
      html += '<div class="accordion-header" onclick="toggleAccordion(this)">';
      html += '<div class="accordion-icon"><svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 9l9-7 9 7v11a2 2 0 01-2 2H5a2 2 0 01-2-2z"/><polyline points="9 22 9 12 15 12 15 22"/></svg></div>';
      html += '<div class="accordion-info"><div class="accordion-city">' + s.city + '</div><div class="accordion-region">' + s.region + '</div></div>';
      html += '<div class="accordion-arrow"><svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M5 7l5 5 5-5"/></svg></div>';
      html += '</div>';
      html += '<div class="accordion-content"><div class="accordion-body">';
      html += '<div class="info-row"><div class="info-row-icon"><svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0118 0z"/><circle cx="12" cy="10" r="3"/></svg></div><div class="info-row-content"><div class="info-row-label">Адрес</div><div class="info-row-value">' + s.address + '</div></div></div>';
      html += '<div class="info-row"><div class="info-row-icon"><svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><path d="M12 6v6l4 2"/></svg></div><div class="info-row-content"><div class="info-row-label">Часы работы</div><div class="info-row-value">' + s.hours + '</div></div></div>';
      html += '<div class="info-row"><div class="info-row-icon"><svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M22 16.92v3a2 2 0 01-2.18 2 19.79 19.79 0 01-8.63-3.07 19.5 19.5 0 01-6-6 19.79 19.79 0 01-3.07-8.67A2 2 0 014.11 2h3a2 2 0 012 1.72 12.84 12.84 0 00.7 2.81 2 2 0 01-.45 2.11L8.09 9.91a16 16 0 006 6l1.27-1.27a2 2 0 012.11-.45 12.84 12.84 0 002.81.7A2 2 0 0122 16.92z"/></svg></div><div class="info-row-content"><div class="info-row-label">Телефон</div><div class="info-row-value">' + s.phone + '</div></div></div>';
      html += '<a href="tel:' + phoneHref + '" class="call-btn">📞 Позвонить</a>';
      if (s.phoneAlt) {
        var altHref = s.phoneAlt.replace(/[+\s\(\)]/g, '').replace(/-/g, '');
        html += '<a href="tel:' + altHref + '" class="call-btn secondary">📞 Доп. номер</a>';
      }
      html += '</div></div></div>';
    }
    accordion.innerHTML = html;
  }

  window.toggleAccordion = function(header) {
    var item = header.parentElement;
    var isOpen = item.classList.contains('open');
    var all = document.querySelectorAll('.accordion-item');
    for (var i = 0; i < all.length; i++) all[i].classList.remove('open');
    if (!isOpen) {
      item.classList.add('open');
      setTimeout(function() { item.scrollIntoView({ behavior: 'smooth', block: 'nearest' }); }, 100);
    }
  };

  var dirSearch = document.getElementById('directorySearch');
  if (dirSearch) {
    dirSearch.addEventListener('input', function() { buildDirectory(this.value); });
  }

  window.openMap = function(type) {
    var modal = document.getElementById('mapModal');
    modal.classList.add('active');
    var locations = {
      departure: { name: 'Место отправления', address: 'Ул. Шамшурина, 43А, Новосибирск', lat: 55.0415, lng: 82.9345 },
      arrival: { name: 'Место назначения', address: 'Ул. Ленина, 81, Славгород', lat: 52.9945, lng: 78.6625 }
    };
    var loc = locations[type];
    document.getElementById('mapModalTitle').textContent = loc.name;
    document.getElementById('mapInfoBar').textContent = '📍 ' + loc.address;
    setTimeout(function() {
      if (mapInstance) { mapInstance.remove(); mapInstance = null; }
      try {
        mapInstance = L.map('map', { zoomControl: true, attributionControl: false }).setView([loc.lat, loc.lng], 15);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 19 }).addTo(mapInstance);
        var color = type === 'departure' ? '#00B87C' : '#1A2B49';
        var icon = L.divIcon({ className: 'custom-marker', html: '<div style="width: 24px; height: 24px; background: ' + color + '; border: 3px solid white; border-radius: 50%; box-shadow: 0 2px 8px rgba(0,0,0,0.3);"></div>', iconSize: [24, 24], iconAnchor: [12, 12] });
        L.marker([loc.lat, loc.lng], { icon: icon }).addTo(mapInstance).bindPopup('<b>' + loc.name + '</b><br>' + loc.address).openPopup();
        setTimeout(function() { mapInstance.invalidateSize(); }, 100);
      } catch(e) { console.log('Map error:', e); }
    }, 300);
  };

  window.closeMap = function() { document.getElementById('mapModal').classList.remove('active'); };
  document.getElementById('mapModal').addEventListener('click', function(e) { if (e.target === this) closeMap(); });

  function initSiberiaMap() {
    if (siberiaMapInstance) {
      siberiaMapInstance.invalidateSize();
      return;
    }
    try {
      siberiaMapInstance = L.map('siberiaMap', { zoomControl: true, attributionControl: false }).setView([54.5, 82], 6);
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 19 }).addTo(siberiaMapInstance);
      
      for (var i = 0; i < busStations.length; i++) {
        (function(station) {
          var isMain = (station.city === 'Новосибирск' || station.city === 'Кемерово' || station.city === 'Омск' || station.city === 'Барнаул');
          var color = isMain ? '#00B87C' : '#00D68F';
          var size = isMain ? 14 : 10;
          var icon = L.divIcon({
            className: 'custom-marker',
            html: '<div style="width: ' + size + 'px; height: ' + size + 'px; background: ' + color + '; border: 3px solid white; border-radius: 50%; box-shadow: 0 2px 8px rgba(0,0,0,0.3);"></div>',
            iconSize: [size, size],
            iconAnchor: [size/2, size/2]
          });
          L.marker([station.lat, station.lng], { icon: icon })
            .addTo(siberiaMapInstance)
            .bindPopup('<b>' + station.city + '</b><br>' + station.region + '<br>' + station.address + '<br>📞 ' + station.phone);
        })(busStations[i]);
      }
      
      setTimeout(function() { siberiaMapInstance.invalidateSize(); }, 200);
    } catch(e) { console.log('Siberia map error:', e); }
  }

  window.openRequestModal = function() { document.getElementById('requestModal').classList.add('active'); };
  window.submitRequest = function() {
    var from = document.getElementById('reqFrom').value;
    var to = document.getElementById('reqTo').value;
    var date = document.getElementById('reqDate').value;
    if (!from || !to || !date) { showToast('Заполните откуда, куда и дату'); return; }
    if (state.requestType === 'parcel') {
      var cargoType = document.getElementById('reqCargoType').value;
      if (cargoType === 'custom') {
        var customCargo = document.getElementById('reqCustomCargo').value;
        if (!customCargo) { showToast('Укажите тип груза'); return; }
      }
      var value = document.getElementById('reqCargoValue').value;
      var recipient = document.getElementById('reqRecipient').value;
      if (!value || value <= 0) { showToast('Укажите объявленную ценность'); return; }
      if (!recipient) { showToast('Укажите получателя'); return; }
    }
    showToast('Заявка опубликована! Водители увидят её.');
    document.getElementById('requestModal').classList.remove('active');
    document.getElementById('reqFrom').value = '';
    document.getElementById('reqTo').value = '';
    document.getElementById('reqComment').value = '';
  };

  window.respondToRequest = function(name, route) {
    showToast('Вы откликнулись на заявку ' + name + ' (' + route + ')');
  };

  window.filterRequests = function(el, type) {
    var tabs = el.parentElement.querySelectorAll('.tab');
    for (var i = 0; i < tabs.length; i++) tabs[i].classList.remove('active');
    el.classList.add('active');
    showToast('Фильтр: ' + (type === 'all' ? 'Все заявки' : type === 'today' ? 'Сегодня' : 'Неделя'));
  };

  window.openAccessModal = function() { document.getElementById('accessModal').classList.add('active'); };
  window.closeAccessModal = function() { document.getElementById('accessModal').classList.remove('active'); };
  document.getElementById('accessModal').addEventListener('click', function(e) { if (e.target === this) closeAccessModal(); });

  window.setFontSize = function(size, btn) {
    document.documentElement.style.setProperty('--base-font-size', size + 'px');
    accessSettings.fontSize = size;
    var btns = document.querySelectorAll('.access-font-btn');
    for (var i = 0; i < btns.length; i++) btns[i].classList.remove('active');
    btn.classList.add('active');
    var label = size === 16 ? 'Обычный' : size === 20 ? 'Крупный' : 'Очень крупный';
    showToast('Размер текста: ' + label);
  };

  window.toggleContrast = function() {
    accessSettings.contrast = !accessSettings.contrast;
    document.body.classList.toggle('high-contrast', accessSettings.contrast);
    document.getElementById('contrastToggle').classList.toggle('active', accessSettings.contrast);
  };

  window.toggleVoice = function() {
    accessSettings.voice = !accessSettings.voice;
    document.getElementById('voiceToggle').classList.toggle('active', accessSettings.voice);
    if (!accessSettings.voice && 'speechSynthesis' in window) {
      window.speechSynthesis.cancel();
    }
  };

  window.toggleSubtitle = function() {
    accessSettings.subtitle = !accessSettings.subtitle;
    document.getElementById('subtitleToggle').classList.toggle('active', accessSettings.subtitle);
    if (!accessSettings.subtitle) {
      document.getElementById('subtitleBar').classList.remove('show');
    }
  };

  window.toggleBigButtons = function() {
    accessSettings.bigButtons = !accessSettings.bigButtons;
    document.getElementById('bigButtonsToggle').classList.toggle('active', accessSettings.bigButtons);
    showToast(accessSettings.bigButtons ? 'Увеличенные кнопки включены' : 'Увеличенные кнопки выключены');
  };

  window.togglePhoneVisibility = function() {
    state.phoneVisibility = !state.phoneVisibility;
    var toggle = document.getElementById('phoneVisibilityToggle');
    toggle.classList.toggle('active', state.phoneVisibility);
    showToast(state.phoneVisibility ? 'Номер телефона будет показан' : 'Номер телефона скрыт');
  };

  window.openAiModal = function() { 
    document.getElementById('aiModal').classList.add('active');
    if ('speechSynthesis' in window && !speechUnlocked) {
      var unlockUtterance = new SpeechSynthesisUtterance('');
      unlockUtterance.volume = 0;
      window.speechSynthesis.speak(unlockUtterance);
      speechUnlocked = true;
    }
  };
  window.closeAiModal = function() { 
    stopListening();
    document.getElementById('aiModal').classList.remove('active'); 
  };
  document.getElementById('aiModal').addEventListener('click', function(e) { if (e.target === this) closeAiModal(); });

  window.openChat = function(name) {
    showToast('Чат с ' + name + ' открыт. Напишите сообщение!');
  };

  window.updateSpeechSpeed = function() {
    var select = document.getElementById('speechSpeed');
    accessSettings.speechSpeed = parseFloat(select.value);
    showToast('Скорость речи: ' + (accessSettings.speechSpeed === 0.7 ? 'Медленно' : accessSettings.speechSpeed === 0.9 ? 'Нормально' : 'Быстро'));
  };

  function addAiMessage(text, type) {
    var messages = document.getElementById('aiMessages');
    var now = new Date();
    var time = ('0' + now.getHours()).slice(-2) + ':' + ('0' + now.getMinutes()).slice(-2);
    var msg = document.createElement('div');
    msg.className = 'ai-message ' + type;
    msg.innerHTML = text + '<span class="msg-time">' + time + '</span>';
    messages.appendChild(msg);
    messages.scrollTop = messages.scrollHeight;
    
    if (type === 'bot' && accessSettings.subtitle) {
      showSubtitle(text);
    }
    
    if (accessSettings.voice && type === 'bot') {
      speakText(text);
    }
  }

  function showSubtitle(text) {
    var cleanText = text.replace(/[👋🔍👥🚗❓📱💰🛡️✅💳🧳📞📝🗺️📏⏱️🏔️🤔👇😊🚗📍📅👤⏰🙋🤖📦📲📋✉️]/g, '').replace(/\n/g, ' ').replace(/<[^>]*>/g, '');
    var bar = document.getElementById('subtitleBar');
    var textEl = document.getElementById('subtitleText');
    textEl.textContent = cleanText;
    bar.classList.add('show');
    if (subtitleTimeout) clearTimeout(subtitleTimeout);
    subtitleTimeout = setTimeout(function() {
      bar.classList.remove('show');
    }, 8000);
  }

  function showTyping() {
    var messages = document.getElementById('aiMessages');
    var typing = document.createElement('div');
    typing.className = 'ai-typing';
    typing.id = 'aiTyping';
    typing.innerHTML = '<span></span><span></span><span></span>';
    messages.appendChild(typing);
    messages.scrollTop = messages.scrollHeight;
  }

  function removeTyping() { var t = document.getElementById('aiTyping'); if (t) t.remove(); }

  // ============ NLU ENGINE ============
  function parseIntent(text) {
    var lower = text.toLowerCase();
    
    // Greeting
    if (lower.match(/^(привет|здравствуй|хай|добрый|салют)/)) {
      return { intent: 'greeting', confidence: 0.95 };
    }
    
    // Search ride
    if (lower.match(/найди|ищу|хочу поехать|мне нужно.*поехать|поездк|отвези|подвези|доехать/)) {
      return { intent: 'search_ride', confidence: 0.9 };
    }
    
    // Create request
    if (lower.match(/оставь|создай|заявк|забронируй|хочу оставить/)) {
      return { intent: 'create_request', confidence: 0.9 };
    }
    
    // Cancel
    if (lower.match(/отмен|удали|отменить/)) {
      return { intent: 'cancel', confidence: 0.9 };
    }
    
    // Price
    if (lower.match(/цен|стоит|стоимост|сколько.*стоит|сколько.*денег/)) {
      return { intent: 'price', confidence: 0.9 };
    }
    
    // Help
    if (lower.match(/помощь|помоги|как работ|что умеешь|расскажи/)) {
      return { intent: 'help', confidence: 0.9 };
    }
    
    // Support
    if (lower.match(/поддержк|оператор|живой|позвонить.*поддержк/)) {
      return { intent: 'support', confidence: 0.9 };
    }
    
    // Goodbye
    if (lower.match(/пока|до свидания|bye|всего хорошего/)) {
      return { intent: 'goodbye', confidence: 0.9 };
    }
    
    // Thanks
    if (lower.match(/спасибо|благодар/)) {
      return { intent: 'thanks', confidence: 0.9 };
    }
    
    // Confirm (for multi-turn)
    if (lower.match(/да|подтверждаю|согласен|верно|правильно|ок|okay/)) {
      return { intent: 'confirm', confidence: 0.85 };
    }
    
    // Deny
    if (lower.match(/нет|не надо|отмена|отменить|не хочу/)) {
      return { intent: 'deny', confidence: 0.85 };
    }
    
    // Parcels
    if (lower.match(/посылк|документ|груз|отправит.*посылк|переслать/)) {
      return { intent: 'parcel', confidence: 0.9 };
    }
    
    return { intent: 'unknown', confidence: 0 };
  }

  function extractEntities(text) {
    var lower = text.toLowerCase();
    var entities = {};
    
    // Cities
    var foundCities = [];
    for (var i = 0; i < cities.length; i++) {
      if (lower.indexOf(cities[i].name.toLowerCase()) !== -1) {
        foundCities.push(cities[i].name);
      }
    }
    if (foundCities.length >= 1) entities.from = foundCities[0];
    if (foundCities.length >= 2) entities.to = foundCities[1];
    
    // Dates
    if (lower.indexOf('завтра') !== -1) entities.date = 'завтра';
    else if (lower.indexOf('послезавтра') !== -1) entities.date = 'послезавтра';
    else if (lower.indexOf('сегодня') !== -1) entities.date = 'сегодня';
    
    // Time
    if (lower.match(/утр|утром|до 12/)) entities.time = 'утро';
    else if (lower.match(/дн|днём|с 12 до 17/)) entities.time = 'день';
    else if (lower.match(/вечер|вечером|после 17/)) entities.time = 'вечер';
    
    // Seats
    var seatsMatch = lower.match(/(\d+)\s*мест/);
    if (seatsMatch) entities.seats = parseInt(seatsMatch[1]);
    else {
      var singleSeatMatch = lower.match(/(\d+)\s*(пассажир|человек|людей)/);
      if (singleSeatMatch) entities.seats = parseInt(singleSeatMatch[1]);
    }
    
    // Parcel type
    if (lower.indexOf('документ') !== -1) entities.cargoType = 'Документы';
    else if (lower.indexOf('коробк') !== -1) entities.cargoType = 'Коробка';
    else if (lower.indexOf('хрупк') !== -1) entities.cargoType = 'Хрупкое';
    else if (lower.indexOf('одежд') !== -1) entities.cargoType = 'Одежда';
    else if (lower.indexOf('электроник') !== -1) entities.cargoType = 'Электроника';
    else if (lower.indexOf('продукт') !== -1) entities.cargoType = 'Продукты';
    
    return entities;
  }

  function handleIntent(intent, entities, originalText) {
    var lower = originalText.toLowerCase();
    
    // Multi-turn dialog handling
    if (dialogState.active) {
      return handleMultiTurn(intent, entities, originalText);
    }
    
    switch(intent) {
      case 'greeting':
        return 'Здравствуйте! 👋 Рад вас видеть. Я ваш голосовой помощник. Скажите, что вам нужно: найти поездку, оставить заявку или узнать цены?';
      
      case 'search_ride':
        return handleSearchRide(entities);
      
      case 'create_request':
        return startCreateRequest(entities);
      
      case 'cancel':
        return 'Чтобы отменить заявку, перейдите в раздел "Ваши поездки" и нажмите на нужную поездку. Там будет кнопка отмены. Хотите, я помогу найти ваши поездки?';
      
      case 'price':
        return handlePriceQuery(entities);
      
      case 'help':
        return '📱 Я могу помочь вам:\n\n' +
          '🔍 Найти поездку — скажите "Найди поездку завтра в Кемерово"\n' +
          '📝 Оставить заявку — скажите "Оставь заявку на 2 места"\n' +
          '💰 Узнать цены — скажите "Сколько стоит поездка в Барнаул"\n' +
          '📦 Отправить посылку — скажите "Хочу отправить посылку"\n' +
          '❓ Задать вопрос — просто спросите!\n\n' +
          'Что вас интересует?';
      
      case 'support':
        return '📞 Для связи с поддержкой:\n\n' +
          'Единая справочная: 8 (800) 555-35-35\n\n' +
          'Или нажмите кнопку "Позвонить в поддержку" ниже.';
      
      case 'parcel':
        return '📦 Для отправки посылки:\n\n' +
          '1. Переключитесь на вкладку "Посылки" в поиске\n' +
          '2. Выберите тип груза: Документы, Коробка, Хрупкое, Одежда, Электроника, Продукты\n' +
          '3. Укажите вес, габариты и объявленную ценность\n\n' +
          'Хотите, я помогу создать заявку на отправку?';
      
      case 'goodbye':
        return 'До свидания! 👋 Хорошего дня и удачной поездки! Если что — я всегда на связи.';
      
      case 'thanks':
        return 'Пожалуйста! 😊 Рад был помочь. Обращайтесь ещё!';
      
      default:
        // Try to extract cities even from unknown intent
        if (entities.from || entities.to) {
          return handleSearchRide(entities);
        }
        return 'Извините, я не совсем понял. 😊 Попробуйте сказать проще:\n\n' +
          '• "Найди поездку завтра в Кемерово"\n' +
          '• "Оставь заявку на 2 места"\n' +
          '• "Сколько стоит поездка в Омск"\n\n' +
          'Или нажмите на одну из кнопок ниже.';
    }
  }

  function handleSearchRide(entities) {
    var from = entities.from || 'Новосибирск';
    var to = entities.to || null;
    var date = entities.date || 'сегодня';
    var time = entities.time || null;
    
    // Find matching rides
    var matches = [];
    for (var i = 0; i < availableRides.length; i++) {
      var ride = availableRides[i];
      var match = true;
      if (to && ride.to.toLowerCase().indexOf(to.toLowerCase()) === -1 && ride.from.toLowerCase().indexOf(to.toLowerCase()) === -1) {
        match = false;
      }
      if (from && ride.from.toLowerCase().indexOf(from.toLowerCase()) === -1 && ride.to.toLowerCase().indexOf(from.toLowerCase()) === -1) {
        match = false;
      }
      if (match) matches.push(ride);
    }
    
    if (matches.length === 0) {
      // Fallback: show any ride
      matches = [availableRides[0]];
    }
    
    var response = '🔍 Нашёл ' + matches.length + ' поездку(и):\n\n';
    for (var j = 0; j < matches.length && j < 3; j++) {
      var r = matches[j];
      response += (j + 1) + '. ' + r.from + ' → ' + r.to + '\n';
      response += '   👤 ' + r.driver + ' (' + r.rating + ')\n';
      response += '   🕐 ' + r.date + ', ' + r.time + '\n';
      response += '   💰 ' + r.price + '\n';
      response += '   📞 ' + r.phone + '\n\n';
    }
    
    response += 'Хотите позвонить водителю или открыть details поездки?';
    return response;
  }

  function handlePriceQuery(entities) {
    var to = entities.to || null;
    var response = '💰 Средние цены по направлениям:\n\n';
    
    if (to && to.toLowerCase().indexOf('славгород') !== -1) {
      response += 'Новосибирск — Славгород: 1500-2000 ₽\n';
    } else if (to && to.toLowerCase().indexOf('кемерово') !== -1) {
      response += 'Новосибирск — Кемерово: 1000-1400 ₽\n';
    } else if (to && to.toLowerCase().indexOf('омск') !== -1) {
      response += 'Новосибирск — Омск: 1200-1600 ₽\n';
    } else if (to && to.toLowerCase().indexOf('барнаул') !== -1) {
      response += 'Новосибирск — Барнаул: 1100-1500 ₽\n';
      response += 'Барнаул — Горно-Алтайск: 800-1200 ₽\n';
    } else {
      response += '• Новосибирск — Славгород: 1500-2000 ₽\n';
      response += '• Кемерово — Новосибирск: 1000-1400 ₽\n';
      response += '• Омск — Новосибирск: 1200-1600 ₽\n';
      response += '• Барнаул — Горно-Алтайск: 800-1200 ₽\n';
      response += '• Бийск — Барнаул: 400-600 ₽\n';
    }
    
    response += '\nЦена зависит от расстояния и комфорта автомобиля. Хотите найти конкретную поездку?';
    return response;
  }

  function startCreateRequest(entities) {
    dialogState.active = true;
    dialogState.intent = 'create_request';
    dialogState.step = 0;
    dialogState.data = entities;
    
    if (entities.from) {
      dialogState.step = 1;
      return 'Отлично! Вы хотите поехать из ' + entities.from + '. А куда вы направляетесь?';
    }
    
    return '📝 Давайте создадим заявку. Скажите, откуда вы едете?';
  }

  function handleMultiTurn(intent, entities, originalText) {
    var lower = originalText.toLowerCase();
    
    if (intent === 'deny' || intent === 'cancel') {
      dialogState.active = false;
      dialogState.data = {};
      return 'Хорошо, отменил создание заявки. Чем ещё могу помочь?';
    }
    
    if (dialogState.intent === 'create_request') {
      switch(dialogState.step) {
        case 0: // From
          if (entities.from) {
            dialogState.data.from = entities.from;
            dialogState.step = 1;
            return 'Отлично! Вы едете из ' + entities.from + '. А куда направляетесь?';
          } else {
            return 'Извините, я не расслышал город отправления. Повторите, пожалуйста: откуда вы едете?';
          }
        
        case 1: // To
          if (entities.to) {
            dialogState.data.to = entities.to;
            dialogState.step = 2;
            return 'Понял, ' + dialogState.data.from + ' → ' + entities.to + '. На какую дату вам нужна поездка? Скажите "сегодня", "завтра" или "послезавтра".';
          } else if (entities.from && !dialogState.data.to) {
            // User said another city
            dialogState.data.to = entities.from;
            dialogState.step = 2;
            return 'Понял, ' + dialogState.data.from + ' → ' + entities.from + '. На какую дату? Скажите "сегодня", "завтра" или "послезавтра".';
          } else {
            return 'Извините, я не расслышал город назначения. Куда вы хотите поехать?';
          }
        
        case 2: // Date
          if (entities.date) {
            dialogState.data.date = entities.date;
            dialogState.step = 3;
            return 'На ' + entities.date + '. Сколько мест вам нужно? Скажите число, например "2 места".';
          } else {
            return 'Извините, я не расслышал дату. Скажите "сегодня", "завтра" или "послезавтра".';
          }
        
        case 3: // Seats
          if (entities.seats) {
            dialogState.data.seats = entities.seats;
            dialogState.step = 4;
            return 'Отлично! ' + entities.seats + ' ' + (entities.seats === 1 ? 'место' : entities.seats < 5 ? 'места' : 'мест') + '. Подтверждаю заявку:\n\n' +
              '📍 ' + dialogState.data.from + ' → ' + dialogState.data.to + '\n' +
              '📅 ' + dialogState.data.date + '\n' +
              '👥 ' + entities.seats + ' ' + (entities.seats === 1 ? 'место' : entities.seats < 5 ? 'места' : 'мест') + '\n\n' +
              'Подтвердить заявку? Скажите "да" или "нет".';
          } else {
            return 'Сколько мест вам нужно? Скажите число, например "2 места" или "1 место".';
          }
        
        case 4: // Confirm
          if (intent === 'confirm') {
            // Create the request
            dialogState.active = false;
            var data = dialogState.data;
            dialogState.data = {};
            
            // Simulate creation
            setTimeout(function() {
              showToast('✅ Заявка создана: ' + data.from + ' → ' + data.to + ', ' + data.date + ', ' + data.seats + ' мест');
            }, 500);
            
            return '✅ Отлично! Заявка создана. Водители увидят её и смогут с вами связаться. Номер телефона указан в вашем профиле. Чем ещё могу помочь?';
          } else if (intent === 'deny') {
            dialogState.active = false;
            dialogState.data = {};
            return 'Хорошо, отменил создание заявки. Чем ещё могу помочь?';
          } else {
            return 'Скажите "да" чтобы подтвердить или "нет" чтобы отменить.';
          }
      }
    }
    
    return 'Извините, произошла ошибка. Давайте начнём сначала. Чем могу помочь?';
  }

  window.sendAiMessage = function() {
    var input = document.getElementById('aiInput');
    var text = input.value.trim();
    if (!text) return;
    addAiMessage(text, 'user');
    input.value = '';
    showTyping();
    setTimeout(function() {
      removeTyping();
      processUserInput(text);
    }, 600 + Math.random() * 400);
  };

  window.sendQuickReply = function(text) {
    addAiMessage(text, 'user');
    showTyping();
    setTimeout(function() {
      removeTyping();
      processUserInput(text);
    }, 600);
  };

  function processUserInput(text) {
    var intent = parseIntent(text);
    var entities = extractEntities(text);
    var response = handleIntent(intent.intent, entities, text);
    addAiMessage(response, 'bot');
  }

  // ============ VOICE INPUT ============
  window.toggleVoiceInput = function() {
    if (isListening) {
      stopListening();
      return;
    }
    startListening();
  };

  function startListening() {
    var SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (!SpeechRecognition) {
      showToast('Голосовой ввод не поддерживается. Используйте Chrome или Safari.');
      return;
    }
    
    try {
      recognition = new SpeechRecognition();
      recognition.lang = 'ru-RU';
      recognition.interimResults = false;
      recognition.continuous = false;
      recognition.maxAlternatives = 1;
      
      var btn = document.getElementById('voiceBtn');
      var visualizer = document.getElementById('voiceVisualizer');
      var status = document.getElementById('voiceStatus');
      
      btn.classList.add('recording');
      visualizer.classList.add('active');
      status.textContent = '🎤 Слушаю... Говорите!';
      status.classList.add('listening');
      isListening = true;
      
      recognition.onstart = function() {
        console.log('Speech recognition started');
      };
      
      recognition.onresult = function(event) {
        var transcript = event.results[0][0].transcript;
        var confidence = event.results[0][0].confidence;
        
        stopListeningUI();
        
        if (accessSettings.subtitle) {
          showSubtitle('🎤 ' + transcript);
        }
        
        addAiMessage(transcript, 'user');
        
        // Low confidence - ask to repeat
        if (confidence < 0.6) {
          showTyping();
          setTimeout(function() {
            removeTyping();
            addAiMessage('Извините, я не совсем расслышал. Вы сказали "' + transcript + '"? Повторите, пожалуйста, или напишите текстом.', 'bot');
          }, 500);
        } else {
          showTyping();
          setTimeout(function() {
            removeTyping();
            processUserInput(transcript);
          }, 600);
        }
      };
      
      recognition.onerror = function(event) {
        stopListeningUI();
        var errorMsg = 'Ошибка микрофона';
        if (event.error === 'no-speech') {
          errorMsg = 'Речь не распознана. Попробуйте говорить громче и чётче.';
        } else if (event.error === 'audio-capture') {
          errorMsg = 'Микрофон не найден. Проверьте подключение.';
        } else if (event.error === 'not-allowed') {
          errorMsg = 'Доступ к микрофону запрещён. Разрешите в настройках браузера.';
        } else if (event.error === 'network') {
          errorMsg = 'Проблема с сетью. Проверьте интернет.';
        }
        showToast(errorMsg);
        addAiMessage('😔 ' + errorMsg + '\n\nПопробуйте ещё раз или напишите текстом.', 'bot');
      };
      
      recognition.onend = function() {
        stopListeningUI();
      };
      
      recognition.start();
    } catch(e) {
      stopListeningUI();
      showToast('Не удалось запустить микрофон: ' + e.message);
    }
  }

  function stopListening() {
    if (recognition) {
      try { recognition.stop(); } catch(e) {}
    }
    stopListeningUI();
  }

  function stopListeningUI() {
    var btn = document.getElementById('voiceBtn');
    var visualizer = document.getElementById('voiceVisualizer');
    var status = document.getElementById('voiceStatus');
    
    btn.classList.remove('recording');
    visualizer.classList.remove('active');
    status.textContent = 'Нажмите на микрофон, чтобы начать';
    status.classList.remove('listening');
    isListening = false;
  }

  function speakText(text) {
    if (!('speechSynthesis' in window)) {
      console.log('Speech synthesis not supported');
      return;
    }
    try {
      window.speechSynthesis.cancel();
      
      var cleanText = text.replace(/[👋🔍👥🚗❓📱💰🛡️✅💳🧳📞📝🗺️📏⏱️🏔️🤔👇😊🚗📍📅👤⏰🙋🤖📦📲📋✉️🎤😔😊]/g, '')
        .replace(/\n/g, '. ')
        .replace(/<[^>]*>/g, '')
        .replace(/\s+/g, ' ')
        .trim();
      
      if (!cleanText) return;
      
      var utterance = new SpeechSynthesisUtterance(cleanText);
      utterance.lang = 'ru-RU';
      utterance.rate = accessSettings.speechSpeed;
      utterance.pitch = 1;
      utterance.volume = 1;
      
      utterance.onstart = function() { console.log('Speech started'); };
      utterance.onend = function() { console.log('Speech ended'); };
      utterance.onerror = function(event) { console.log('Speech error:', event.error); };
      
      setTimeout(function() {
        window.speechSynthesis.speak(utterance);
      }, 100);
    } catch(e) {
      console.log('Speech error:', e);
    }
  }

  // Init
  buildCalendar();
  buildDirectory('');
  setTimeout(initSiberiaMap, 500);

})();
</script>
</body>
</html>
