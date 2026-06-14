// ==========================================
// 短信 APP (iMessage) 独立逻辑模块 (完美重构版 v3)
// ==========================================

const smsState = {
    chats: [], // 短信会话列表
    currentSender: localStorage.getItem('ios_sms_current_sender') || '我 (User)', // 👇 修改：从本地存储读取当前马甲
    activeChatId: null,
    currentTab: 'chats'
};

// 1. 动态注入 CSS 样式 (低饱和黑白灰高级风)
const smsStyle = document.createElement('style');
smsStyle.innerHTML = `
    .sms-view-container { position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: none; flex-direction: column; background: #EAEAEA; animation: wcFadeIn 0.3s ease; }
    .sms-view-container.active { display: flex; }
    
    /* Chats 列表页 */
    .sms-chats-header { display: flex; justify-content: space-between; align-items: center; padding: calc(env(safe-area-inset-top, 20px) + 10px) 24px 20px; }
    .sms-chats-header h1 { font-size: 36px; font-weight: 900; color: #111; letter-spacing: -0.5px; cursor: pointer; }
    .sms-search-btn { width: 44px; height: 44px; background: #FFF; border-radius: 50%; display: flex; align-items: center; justify-content: center; box-shadow: 0 4px 10px rgba(0,0,0,0.05); cursor: pointer; }
    .sms-search-btn svg { width: 20px; height: 20px; stroke: #111; stroke-width: 2; fill: none; }

    /* 顶部全部角色横向滚动 */
    .sms-story-scroll { display: flex; gap: 16px; padding: 0 24px 24px; overflow-x: auto; scrollbar-width: none; }
    .sms-story-scroll::-webkit-scrollbar { display: none; }
    .sms-story-item { display: flex; flex-direction: column; align-items: center; flex-shrink: 0; cursor: pointer; transition: transform 0.2s; }
    .sms-story-item:active { transform: scale(0.95); }
    .sms-story-avatar { width: 64px; height: 64px; border-radius: 50%; object-fit: cover; border: 2px solid #FFF; box-shadow: 0 4px 10px rgba(0,0,0,0.05); }
    .sms-story-name { font-size: 11px; color: #555; font-weight: 600; margin-top: 6px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; max-width: 64px; text-align: center; }

    /* 👇 修改：去掉了 padding-bottom: 100px，因为没有底栏了 👇 */
    .sms-chats-body { flex: 1; background: #FFF; border-radius: 32px 32px 0 0; padding: 24px 20px calc(20px + env(safe-area-inset-bottom, 0px)); overflow-y: auto; box-shadow: 0 -4px 20px rgba(0,0,0,0.03); }
    .sms-section-title { font-size: 13px; font-weight: 700; color: #888; margin-bottom: 16px; margin-top: 10px; text-transform: uppercase; letter-spacing: 0.5px; }
    
    .sms-chat-list-item { display: flex; align-items: center; margin-bottom: 24px; cursor: pointer; }
    .sms-chat-list-item:active { opacity: 0.6; }
    .sms-chat-list-avatar-wrap { position: relative; margin-right: 16px; }
    .sms-chat-list-avatar { width: 52px; height: 52px; border-radius: 50%; object-fit: cover; background: #F5F5F5; display: flex; align-items: center; justify-content: center; font-weight: bold; color: #888; }
    .sms-status-dot { position: absolute; bottom: 2px; right: 2px; width: 14px; height: 14px; background: #111; border: 2px solid #FFF; border-radius: 50%; }
    .sms-chat-list-info { flex: 1; overflow: hidden; }
    .sms-chat-list-name-row { display: flex; justify-content: space-between; align-items: center; margin-bottom: 4px; }
    .sms-chat-list-name { font-size: 16px; font-weight: 700; color: #111; }
    .sms-chat-list-time { font-size: 12px; color: #888; }
    .sms-chat-list-msg-row { display: flex; justify-content: space-between; align-items: center; }
    .sms-chat-list-msg { font-size: 14px; color: #888; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; max-width: 200px; }
    .sms-unread-badge { background: #111; color: #FFF; font-size: 10px; font-weight: bold; padding: 2px 6px; border-radius: 10px; }
    .sms-unread-badge.danger { background: #FF3B30; }

    /* 聊天详情页 */
    #sms-view-chat-detail { background: #FFF; z-index: 20; }
    .sms-chat-detail-header { display: flex; align-items: center; justify-content: space-between; padding: calc(env(safe-area-inset-top, 20px) + 10px) 16px 15px; border-bottom: 1px solid #F0F0F0; background: rgba(255,255,255,0.95); backdrop-filter: blur(10px); z-index: 10; }
    .sms-back-btn { display: flex; align-items: center; color: #111; font-size: 16px; cursor: pointer; width: 60px; }
    .sms-back-btn svg { width: 24px; height: 24px; stroke: currentColor; stroke-width: 2; fill: none; margin-right: -4px; }
    .sms-chat-detail-title { display: flex; flex-direction: column; align-items: center; cursor: pointer; transition: opacity 0.2s; }
    .sms-chat-detail-title:active { opacity: 0.5; }
    .sms-chat-detail-avatar { width: 36px; height: 36px; border-radius: 50%; object-fit: cover; margin-bottom: 4px; background: #F5F5F5; display: flex; align-items: center; justify-content: center; font-size: 12px; font-weight: bold; color: #888; }
    .sms-chat-detail-name { font-size: 12px; font-weight: 600; color: #111; display: flex; align-items: center; gap: 4px; }
    .sms-chat-detail-name svg { width: 10px; height: 10px; stroke: #888; fill: none; stroke-width: 2; }
    .sms-chat-detail-right { width: 60px; }

    .sms-chat-messages { flex: 1; overflow-y: auto; padding: 20px 16px; display: flex; flex-direction: column; gap: 16px; }
    .sms-bubble-row { display: flex; flex-direction: column; max-width: 75%; }
    .sms-bubble-row.me { align-self: flex-end; align-items: flex-end; }
    .sms-bubble-row.them { align-self: flex-start; align-items: flex-start; }
    .sms-bubble { padding: 10px 14px; border-radius: 18px; font-size: 15px; line-height: 1.4; word-break: break-word; }
    .sms-bubble-row.me .sms-bubble { background: #222; color: #FFF; border-bottom-right-radius: 4px; }
    .sms-bubble-row.them .sms-bubble { background: #F2F2F7; color: #111; border-bottom-left-radius: 4px; }
    
    .sms-bubble-row.fake-sender { align-self: flex-end; align-items: flex-end; }
    .sms-bubble-row.fake-sender .sms-bubble { background: #222; color: #FFF; border-bottom-right-radius: 4px; border: none; }
    .sms-fake-sender-label { font-size: 11px; color: #888; margin-bottom: 4px; margin-right: 4px; font-family: monospace; text-align: right; }

    .sms-chat-input-area { padding: 10px 16px calc(10px + env(safe-area-inset-bottom, 20px)); background: rgba(255,255,255,0.95); border-top: 1px solid #F0F0F0; display: flex; flex-direction: column; gap: 8px; }
    .sms-sender-indicator { font-size: 11px; color: #888; display: flex; align-items: center; gap: 4px; }
    .sms-sender-indicator span { font-weight: bold; color: #111; background: #EAEAEA; padding: 2px 8px; border-radius: 10px; cursor: pointer; }
    .sms-input-bar { display: flex; align-items: center; gap: 12px; }
    
    .sms-ai-btn { width: 32px; height: 32px; display: flex; align-items: center; justify-content: center; cursor: pointer; color: #AF52DE; }
    .sms-ai-btn svg { width: 22px; height: 22px; fill: currentColor; }

    .sms-input-bar input { flex: 1; height: 36px; border-radius: 18px; border: 1px solid #F0F0F0; padding: 0 16px; font-size: 15px; outline: none; background: #F9F9F9; }
    .sms-send-btn { width: 32px; height: 32px; background: #111; border-radius: 50%; display: flex; align-items: center; justify-content: center; cursor: pointer; }
    .sms-send-btn svg { width: 16px; height: 16px; fill: #FFF; transform: translateX(-1px); }

    /* 全屏角色信息页 */
    #sms-view-char-profile { background: #F2F2F7; z-index: 30; }
    .sms-char-profile-top { padding: calc(env(safe-area-inset-top, 20px) + 40px) 20px 30px; display: flex; flex-direction: column; align-items: center; background: #FFF; border-bottom: 1px solid #EAEAEA; }
    .sms-char-profile-top img { width: 100px; height: 100px; border-radius: 50%; object-fit: cover; margin-bottom: 16px; box-shadow: 0 4px 20px rgba(0,0,0,0.1); }
    .sms-char-profile-top h2 { font-size: 24px; font-weight: 800; color: #111; }
    
    .sms-char-actions { padding: 20px; display: flex; flex-direction: column; gap: 12px; }
    .sms-char-action-btn { background: #FFF; border-radius: 12px; padding: 16px; text-align: center; font-size: 16px; font-weight: 600; cursor: pointer; transition: background 0.2s; }
    .sms-char-action-btn:active { background: #F9F9F9; }
    .sms-char-action-btn.danger { color: #FF3B30; }

    /* 弹窗样式 */
    .sms-modal-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.4); z-index: 3000; display: none; align-items: flex-end; justify-content: center; }
    .sms-modal-overlay.active { display: flex; }
    .sms-action-sheet { width: 100%; background: #F2F2F7; border-radius: 20px 20px 0 0; padding: 20px; animation: slideUp 0.3s ease; }
    @keyframes slideUp { from { transform: translateY(100%); } to { transform: translateY(0); } }
    .sms-sheet-btn { width: 100%; background: #FFF; padding: 16px; text-align: center; font-size: 16px; color: #111; font-weight: 500; border-bottom: 1px solid #F0F0F0; cursor: pointer; }
    .sms-sheet-btn:first-child { border-radius: 12px 12px 0 0; }
    .sms-sheet-btn:last-child { border-radius: 0 0 12px 12px; border-bottom: none; }
    .sms-sheet-btn.cancel { margin-top: 10px; border-radius: 12px; font-weight: 600; }
`;
document.head.appendChild(smsStyle);

// 2. 渲染基础 HTML 结构
function smsRenderRoot() {
    const root = document.getElementById('sms-root');
    if (!root) return;
    
    root.innerHTML = `
        <!-- 视图 1：信息列表 -->
        <div id="sms-view-chats" class="sms-view-container active">
            <div class="sms-chats-header">
                <h1 onclick="closeSmsApp()" title="点击退出">信息</h1>
                <div class="sms-search-btn">
                    <svg viewBox="0 0 24 24"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>
                </div>
            </div>
            <!-- 顶部全部角色横向滚动 -->
            <div class="sms-story-scroll" id="sms-story-scroll"></div>
            <div class="sms-chats-body" id="sms-chats-list-container"></div>
        </div>

        <!-- 视图 2：聊天详情页 -->
        <div id="sms-view-chat-detail" class="sms-view-container">
            <div class="sms-chat-detail-header">
                <div class="sms-back-btn" onclick="smsSwitchTab('chats')">
                    <svg viewBox="0 0 24 24"><polyline points="15 18 9 12 15 6"></polyline></svg>
                </div>
                <div class="sms-chat-detail-title" onclick="smsOpenCharProfile()">
                    <img src="" id="sms-chat-header-avatar" class="sms-chat-detail-avatar">
                    <div class="sms-chat-detail-name">
                        <span id="sms-chat-header-name">Name</span>
                        <svg viewBox="0 0 24 24"><polyline points="9 18 15 12 9 6" transform="rotate(-90 12 12)"></polyline></svg>
                    </div>
                </div>
                <div class="sms-chat-detail-right"></div>
            </div>
            <div class="sms-chat-messages" id="sms-chat-messages-container"></div>
            <div class="sms-chat-input-area" style="background: #FFF;">
                <!-- 👇 恢复显示，并居中放在输入框上方 👇 -->
                <div class="sms-sender-indicator" style="display: flex; justify-content: center; margin-bottom: 8px;">
                    <span id="sms-current-sender-display" onclick="smsOpenSenderSheet()" style="background: #F5F5F5; padding: 4px 12px; border-radius: 12px; font-size: 11px; color: #888; cursor: pointer;">${smsState.currentSender}</span>
                </div>
                <div class="sms-input-bar">
                    <div class="sms-ai-btn" onclick="smsTriggerAI_Manual()" style="width: 32px; height: 32px; background: #F5F5F5; border-radius: 50%; display: flex; align-items: center; justify-content: center; cursor: pointer; color: #111; flex-shrink: 0;">
                        <svg viewBox="0 0 24 24" style="width: 18px; height: 18px; fill: currentColor;"><path d="M12 14c1.66 0 3-1.34 3-3V5c0-1.66-1.34-3-3-3S9 3.34 9 5v6c0 1.66 1.34 3 3 3zm5-3c0 2.76-2.24 5-5 5s-5-2.24-5-5H5c0 3.53 2.61 6.43 6 6.92V21h2v-3.08c3.39-.49 6-3.39 6-6.92h-2z"/></svg>
                    </div>
                    <input type="text" id="sms-chat-input" placeholder="iMessage" onkeypress="if(event.key==='Enter') smsSendMessage()" style="flex: 1; height: 36px; border-radius: 18px; border: 1px solid #F0F0F0; padding: 0 16px; font-size: 15px; outline: none; background: #F9F9F9;">
                    <div onclick="smsSendMessage()" style="background: #111; border-radius: 5px; display: flex; align-items: center; justify-content: center; color: #FFF; font-weight: bold; font-size: 14px; cursor: pointer; padding: 0 16px; height: 36px; flex-shrink: 0;">
                        Endit
                    </div>
                </div>
            </div>
        </div>

        <!-- 视图 3：全屏角色信息页 -->
        <div id="sms-view-char-profile" class="sms-view-container">
            <div class="sms-chats-header" style="padding-bottom: 10px; background: #FFF;">
                <div class="sms-back-btn" onclick="smsSwitchTab('chat-detail')">
                    <svg viewBox="0 0 24 24"><polyline points="15 18 9 12 15 6"></polyline></svg>
                </div>
            </div>
            <div class="sms-char-profile-top">
                <img src="" id="sms-char-profile-avatar">
                <h2 id="sms-char-profile-name">Name</h2>
            </div>
            <div class="sms-char-actions">
                <div class="sms-char-action-btn danger" onclick="smsClearChatHistory()">清空当前会话记录</div>
                <div class="sms-char-action-btn danger" onclick="smsDeleteChat()">删除当前会话</div>
            </div>
        </div>

        <!-- 身份选择 Action Sheet -->
        <div class="sms-modal-overlay" id="sms-sender-sheet-modal" onclick="closeSmsSenderSheet()">
            <div class="sms-action-sheet" onclick="event.stopPropagation()">
                <div style="margin-bottom: 10px; text-align: center; font-size: 13px; color: #888; font-weight: bold;">选择发件人身份</div>
                <div style="border-radius: 12px; overflow: hidden;">
                    <div class="sms-sheet-btn" onclick="smsSelectSender('我 (User)')">我 (User)</div>
                    <div class="sms-sheet-btn" onclick="smsSelectSender('10086')">10086 (广告/话费)</div>
                    <div class="sms-sheet-btn" onclick="smsSelectSender('丰巢驿站')">丰巢驿站</div>
                    <div class="sms-sheet-btn" onclick="smsSelectSender('System')" style="color: #888;">System (系统通知)</div>
                    <div class="sms-sheet-btn" onclick="smsAddCustomSender()" style="color: #007AFF;">+ 添加自定义号码</div>
                </div>
                <div class="sms-sheet-btn cancel" onclick="closeSmsSenderSheet()">取消</div>
            </div>
        </div>
    `;
}

// 3. 打开/关闭 APP
window.openSmsApp = async function() {
    await smsLoadData();
    smsRenderRoot();
    document.getElementById('smsModal').classList.add('open');
    smsSwitchTab('chats');
    smsRenderList();
};

window.closeSmsApp = function() {
    document.getElementById('smsModal').classList.remove('open');
};

// 4. 切换 Tab
window.smsSwitchTab = function(tab) {
    smsState.currentTab = tab;
    document.querySelectorAll('.sms-view-container').forEach(el => el.classList.remove('active'));
    const targetView = document.getElementById('sms-view-' + tab);
    if (targetView) targetView.classList.add('active');
    
    if (tab === 'chats') {
        smsRenderList();
    }
};

// 5. 渲染会话列表 & 顶部全部角色
function smsRenderList() {
    const container = document.getElementById('sms-chats-list-container');
    const storyContainer = document.getElementById('sms-story-scroll');
    if (!container || !storyContainer) return;
    
    // 渲染顶部全部角色 (排除群聊)
    storyContainer.innerHTML = '';
    const allChars = wcState.characters.filter(c => !c.isGroup);
    allChars.forEach(char => {
        storyContainer.innerHTML += `
            <div class="sms-story-item" onclick="smsCreateNewChat(${char.id})">
                <img src="${char.avatar}" class="sms-story-avatar">
                <div class="sms-story-name">${char.name}</div>
            </div>
        `;
    });

    // 渲染下方的会话列表
    container.innerHTML = '<div class="sms-section-title">活跃会话</div>';
    if (smsState.chats.length === 0) {
        container.innerHTML += '<div style="text-align:center; color:#999; padding:40px 0;">暂无短信记录<br><span style="font-size:11px;">点击上方头像可新建会话</span></div>';
        return;
    }

    smsState.chats.forEach(chat => {
        const lastMsg = chat.history.length > 0 ? chat.history[chat.history.length - 1] : null;
        const timeStr = lastMsg ? new Date(lastMsg.time).toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}) : '';
        const msgText = lastMsg ? (lastMsg.sender === 'me' ? `你: ${lastMsg.content}` : lastMsg.content) : '';
        
        let avatarHtml = '';
        if (chat.avatar) {
            avatarHtml = `<img src="${chat.avatar}" class="sms-chat-list-avatar">`;
        } else {
            avatarHtml = `<div class="sms-chat-list-avatar">${chat.name.charAt(0)}</div>`;
        }

        container.innerHTML += `
            <div class="sms-chat-list-item" onclick="smsOpenChat('${chat.id}')">
                <div class="sms-chat-list-avatar-wrap">
                    ${avatarHtml}
                    ${chat.unread > 0 ? '<div class="sms-status-dot"></div>' : ''}
                </div>
                <div class="sms-chat-list-info">
                    <div class="sms-chat-list-name-row">
                        <span class="sms-chat-list-name">${chat.name}</span>
                        <span class="sms-chat-list-time">${timeStr}</span>
                    </div>
                    <div class="sms-chat-list-msg-row">
                        <span class="sms-chat-list-msg" style="${chat.unread > 0 ? 'color:#111; font-weight:500;' : ''}">${msgText}</span>
                        ${chat.unread > 0 ? `<div class="sms-unread-badge ${chat.name === '10086' ? 'danger' : ''}">${chat.unread}</div>` : ''}
                    </div>
                </div>
            </div>
        `;
    });
}

// 6. 新建会话 (支持多开，修复保存问题)
window.smsCreateNewChat = function(charId) {
    const char = wcState.characters.find(c => c.id === charId);
    if (!char) return;
    
    const newChat = {
        id: 'sms_' + Date.now() + '_' + Math.floor(Math.random() * 1000),
        realCharId: char.id,
        name: char.name,
        avatar: char.avatar,
        unread: 0,
        history: []
    };
    
    // 👈 核心修复：直接推入数组并保存，确保空会话也能被记住
    smsState.chats.unshift(newChat);
    smsSaveData();
    
    smsOpenChat(newChat.id);
};

// 7. 打开聊天详情
window.smsOpenChat = function(chatId) {
    smsState.activeChatId = chatId;
    const chat = smsState.chats.find(c => c.id === chatId);
    if (!chat) return;

    chat.unread = 0; 
    smsSaveData();

    document.getElementById('sms-chat-header-name').innerText = chat.name;
    const avatarEl = document.getElementById('sms-chat-header-avatar');
    if (chat.avatar) {
        avatarEl.src = chat.avatar;
        avatarEl.style.display = 'block';
        avatarEl.style.background = 'transparent';
        avatarEl.innerText = '';
    } else {
        avatarEl.src = '';
        avatarEl.style.background = '#F5F5F5';
        avatarEl.innerText = chat.name.charAt(0);
    }

    smsRenderChatDetail();
    smsSwitchTab('chat-detail');
};

// 8. 渲染聊天记录
function smsRenderChatDetail() {
    const container = document.getElementById('sms-chat-messages-container');
    container.innerHTML = '';
    
    const chat = smsState.chats.find(c => c.id === smsState.activeChatId);
    if (!chat || chat.history.length === 0) return;

    let lastTime = 0;
    chat.history.forEach(msg => {
        if (msg.time - lastTime > 5 * 60 * 1000) {
            container.innerHTML += `<div style="text-align: center; font-size: 11px; color: #888; margin-bottom: 10px;">${new Date(msg.time).toLocaleString('zh-CN', {month:'short', day:'numeric', hour:'2-digit', minute:'2-digit'})}</div>`;
            lastTime = msg.time;
        }

        // 👇 新增：绑定长按事件
        const touchEvents = `ontouchstart="handleSmsTouchStart(event, ${msg.id})" ontouchend="handleSmsTouchEnd()" oncontextmenu="showSmsContextMenu(event, ${msg.id})"`;

        if (msg.sender === 'me') {
            if (msg.fakeSender && msg.fakeSender !== '我 (User)') {
                container.innerHTML += `
                    <div class="sms-bubble-row fake-sender">
                        <div class="sms-fake-sender-label">from: ${msg.fakeSender}</div>
                        <div class="sms-bubble" ${touchEvents}>${msg.content}</div>
                    </div>
                `;
            } else {
                container.innerHTML += `
                    <div class="sms-bubble-row me">
                        <div class="sms-bubble" ${touchEvents}>${msg.content}</div>
                    </div>
                `;
            }
        } else {
            container.innerHTML += `
                <div class="sms-bubble-row them">
                    <div class="sms-bubble" ${touchEvents}>${msg.content}</div>
                </div>
            `;
        }
    });
    
    setTimeout(() => { container.scrollTop = container.scrollHeight; }, 50);
}

// 9. 接收来自被拉黑 AI 的短信 (路由到最新会话)
window.smsReceiveMessage = function(charName, charAvatar, content, realCharId) {
    // 寻找该角色最新活跃的会话
    let chat = smsState.chats.find(c => c.realCharId === realCharId);
    
    if (!chat) {
        chat = {
            id: 'sms_' + Date.now(),
            realCharId: realCharId,
            name: charName,
            avatar: charAvatar,
            unread: 0,
            history: []
        };
        smsState.chats.unshift(chat);
    } else {
        // 把它提到最前面
        smsState.chats = smsState.chats.filter(c => c.id !== chat.id);
        smsState.chats.unshift(chat);
    }

    chat.history.push({
        id: Date.now(),
        sender: 'them',
        content: content,
        time: Date.now()
    });
    
    chat.unread += 1;
    smsSaveData();
    
    const isSmsAppOpen = document.getElementById('smsModal').classList.contains('open');
    const isViewingThisChat = isSmsAppOpen && smsState.currentTab === 'chat-detail' && smsState.activeChatId === chat.id;

    if (isSmsAppOpen) {
        if (smsState.currentTab === 'chats') smsRenderList();
        if (isViewingThisChat) {
            chat.unread = 0;
            smsRenderChatDetail();
        }
    }

    // 👇 核心修复：只要不是正在看当前聊天，就弹出通知，并强制提升通知层级
    if (!isViewingThisChat) {
        if (typeof showMainSystemNotification === 'function') {
            const notifContainer = document.getElementById('ios-notification-container');
            if (notifContainer) notifContainer.style.zIndex = '99999'; // 确保盖过短信APP
            
            showMainSystemNotification("新短信", `${charName}: ${content}`, charAvatar);
        }
    }
};

// 10. 用户在短信 APP 中发送消息
window.smsSendMessage = async function() {
    const input = document.getElementById('sms-chat-input');
    const text = input.value.trim();
    if (!text || !smsState.activeChatId) return;

    const chat = smsState.chats.find(c => c.id === smsState.activeChatId);
    if (!chat) return;

    chat.history.push({
        id: Date.now(),
        sender: 'me',
        fakeSender: smsState.currentSender, 
        content: text,
        time: Date.now()
    });

    // 把当前会话提到最前面
    smsState.chats = smsState.chats.filter(c => c.id !== chat.id);
    smsState.chats.unshift(chat);

    input.value = '';
    smsSaveData();
    smsRenderChatDetail();

    // 👇 新增：同步到微信记忆 (包含马甲身份)
    if (chat.realCharId) {
        const char = wcState.characters.find(c => c.id === chat.realCharId);
        if (char) {
            let senderName = smsState.currentSender === '我 (User)' ? 'User' : smsState.currentSender;
            wcAddMessage(char.id, 'system', 'system', `[系统内部信息(仅AI可见): 你刚刚收到了一条来自【${senderName}】的短信/iMessage: "${text}"]`, { hidden: true });
        }
    }
    // 👆 新增结束
};

// 11. 触发 AI 短信回复 (携带马甲设定)
async function smsTriggerAI(charId, userText, fakeSender, targetChatId) {
    const char = wcState.characters.find(c => c.id === charId);
    if (!char) return;

    const apiConfig = await getActiveApiConfig('chat');
    if (!apiConfig || !apiConfig.key) return;

    try {
        const chatConfig = char.chatConfig || {};
        const userName = chatConfig.userName || (typeof wcState !== 'undefined' ? wcState.user.name : 'User');
        const userPersona = chatConfig.userPersona || (typeof wcState !== 'undefined' ? wcState.user.persona : '无');
        
        // 1. 读取关联的世界书
        let wbInfo = "";
        if (typeof worldbookEntries !== 'undefined' && worldbookEntries.length > 0 && chatConfig.worldbookEntries && chatConfig.worldbookEntries.length > 0) {
            const linkedEntries = worldbookEntries.filter(e => chatConfig.worldbookEntries.includes(e.id.toString()));
            if (linkedEntries.length > 0) {
                wbInfo = "【附加设定和内容补充参考(世界书)】:\n" + linkedEntries.map(e => `${e.title}: ${e.desc}`).join('\n') + "\n\n";
            }
        }

        // 2. 读取记忆与潜意识
        let memoryText = "暂无特殊记忆。";
        if (char.memories && char.memories.length > 0) {
            const readCount = chatConfig.aiMemoryCount || 5;
            memoryText = char.memories.slice(0, readCount).map(m => {
                // 调用微信的记忆格式化函数，剥离多余标签
                return typeof formatMemoryForAI === 'function' ? `- ${formatMemoryForAI(m.content).replace(/^\[.*?\]\s*/, '')}` : `- ${m.content}`;
            }).join('\n');
        }

        // 3. 读取上下文条数限制
        const contextLimit = (chatConfig.contextLimit > 0) ? chatConfig.contextLimit : 30;

        // 4. 提取微信聊天记录
        let recentWechatMsgs = "暂无微信聊天记录。";
        if (typeof wcState !== 'undefined' && wcState.chats[charId]) {
            const msgs = wcState.chats[charId];
            recentWechatMsgs = msgs.filter(m => !m.isError && m.type !== 'system')
                                 .slice(-contextLimit)
                                 .map(m => {
                                     let content = m.content;
                                     if (m.type !== 'text') content = `[${m.type}]`;
                                     return `${m.sender === 'me' ? userName : char.name}: ${content}`;
                                 })
                                 .join('\n');
        }

        // 5. 提取短信聊天记录 (同样受 contextLimit 限制)
        const smsChat = smsState.chats.find(c => c.id === targetChatId);
        let recentSmsMsgs = "暂无短信历史。";
        if (smsChat && smsChat.history && smsChat.history.length > 0) {
            // 排除当前刚刚发出的这条消息，取之前的 contextLimit 条
            const historyToRead = smsChat.history.slice(-(contextLimit + 1), -1); 
            if (historyToRead.length > 0) {
                recentSmsMsgs = historyToRead.map(m => {
                    const senderName = m.sender === 'me' ? (m.fakeSender || userName) : char.name;
                    return `${senderName}: ${m.content}`;
                }).join('\n');
            }
        }

        // 6. 组装终极 Prompt
        let prompt = `你扮演角色：${char.name}。\n人设：${char.prompt}\n\n`;
        prompt += wbInfo;
        prompt += `【用户(${userName})设定/面具】：${userPersona}\n\n`;
        prompt += `【你们的共同记忆】：\n${memoryText}\n\n`;
        prompt += `【你们最近在微信上的聊天记录（作为前情提要）】：\n${recentWechatMsgs}\n\n`;
        prompt += `【你们最近的短信记录】：\n${recentSmsMsgs}\n\n`;

        // 7. 动态判断是否真的被拉黑
        if (char.isBlocked) {
            prompt += `【核心情境警告】：你目前在微信上被 ${userName} 拉黑了！你现在只能通过【手机短信】接收和发送消息。\n`;
        } else {
            prompt += `【当前情境】：你正在和 ${userName} 通过【手机短信/iMessage】进行聊天。\n`;
        }
        
        if (fakeSender === '我 (User)' || fakeSender === userName) {
            prompt += `${userName} 刚刚通过短信给你发了一条消息：“${userText}”。\n`;
        } else {
            prompt += `你刚刚收到了一条来自【${fakeSender}】的短信：“${userText}”。\n`;
            prompt += `注意：这可能是 ${userName} 伪装的，也可能是真实的系统/广告短信，请根据你的人设做出真实的反应（比如疑惑、回复退订、或者识破是 ${userName}）。\n`;
        }

        // 8. 读取微信聊天设置中的气泡数限制
        const rMin = (chatConfig.replyMin !== undefined) ? chatConfig.replyMin : 1;
        const rMax = (chatConfig.replyMax !== undefined) ? chatConfig.replyMax : 3;

        prompt += `请直接输出你的短信回复内容（纯文本，不要 JSON，不要动作描写，就像真实的短信一样简短）。\n`;
        prompt += `【碎片化口语化强制指令】：必须像真人聊天一样，将长回复拆分成 ${rMin}-${rMax} 条短消息！严禁把所有话挤在一个气泡里！每条短消息之间请用换行符分隔。\n`;

        const response = await fetch(`${apiConfig.baseUrl}/chat/completions`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json', 'Authorization': `Bearer ${apiConfig.key}` },
            body: JSON.stringify({
                model: apiConfig.model,
                messages: [{ role: "user", content: prompt }],
                temperature: 0.8
            })
        });

        const data = await response.json();
        let replyText = data.choices[0].message.content.replace(/<thinking>[\s\S]*?<\/thinking>/g, '').trim();

        // 👈 核心修复：将 AI 的回复精准推入触发它的那个会话中
        const chat = smsState.chats.find(c => c.id === targetChatId);
        if (chat) {
            // 👇 新增：按换行符拆分气泡
            const replies = replyText.split('\n').filter(line => line.trim() !== '');
            let combinedReply = "";
            
            for (const reply of replies) {
                chat.history.push({
                    id: Date.now() + Math.random(),
                    sender: 'them',
                    content: reply.trim(),
                    time: Date.now()
                });
                combinedReply += reply.trim() + " ";
            }
            
            // 提到最前面
            smsState.chats = smsState.chats.filter(c => c.id !== chat.id);
            smsState.chats.unshift(chat);
            
            smsSaveData();
            
            if (smsState.currentTab === 'chat-detail' && smsState.activeChatId === chat.id) {
                smsRenderChatDetail();
            } else {
                chat.unread += replies.length;
                if (smsState.currentTab === 'chats') smsRenderList();
            }

            // 👇 新增：同步到微信记忆
            if (char) {
                wcAddMessage(char.id, 'system', 'system', `[系统内部信息(仅AI可见): 你刚刚通过短信/iMessage回复了 User: "${combinedReply.trim()}"]`, { hidden: true });
            }
            // 👆 新增结束
        }

    } catch (e) {
        console.error("短信 AI 回复失败", e);
    }
}

// 👈 新增：手动点击魔法棒触发 AI 回复
window.smsTriggerAI_Manual = async function() {
    const chat = smsState.chats.find(c => c.id === smsState.activeChatId);
    if (!chat || !chat.realCharId) return alert("该会话无法使用 AI 回复哦~");
    
    // 找到用户最后发的一条消息
    let lastUserText = "";
    let lastFakeSender = "我 (User)";
    for (let i = chat.history.length - 1; i >= 0; i--) {
        if (chat.history[i].sender === 'me') {
            lastUserText = chat.history[i].content;
            lastFakeSender = chat.history[i].fakeSender || "我 (User)";
            break;
        }
    }

    // 插入 loading 气泡
    const container = document.getElementById('sms-chat-messages-container');
    container.innerHTML += `<div class="sms-bubble-row them" id="sms-loading"><div class="sms-bubble" style="color:#888;">正在输入...</div></div>`;
    container.scrollTop = container.scrollHeight;

    await smsTriggerAI(chat.realCharId, lastUserText, lastFakeSender, chat.id);
    
    const loadingEl = document.getElementById('sms-loading');
    if (loadingEl) loadingEl.remove();
};

// 12. 10086 话费不足提醒
window.checkApiQuotaForSms = function(balanceStr) {
    if (balanceStr.includes('$')) {
        const balance = parseFloat(balanceStr.replace('$', ''));
        if (!isNaN(balance) && balance < 0.5) {
            smsReceiveMessage(
                "10086", 
                "", 
                `【余额提醒】尊敬的用户，您的 API 账户余额已不足 $0.50 (当前 ${balanceStr})，为避免影响您的正常聊天服务，请及时前往通讯录节点充值。`, 
                "system_10086"
            );
        }
    }
};
// 14. 全屏角色信息页与清理逻辑
window.smsOpenCharProfile = function() {
    const chat = smsState.chats.find(c => c.id === smsState.activeChatId);
    if (!chat) return;
    
    document.getElementById('sms-char-profile-avatar').src = chat.avatar || '';
    document.getElementById('sms-char-profile-name').innerText = chat.name;
    
    smsSwitchTab('char-profile');
};

window.smsClearChatHistory = function() {
    if(confirm("确定要清空当前会话的短信记录吗？")) {
        const chat = smsState.chats.find(c => c.id === smsState.activeChatId);
        if (chat) {
            chat.history = [];
            smsSaveData();
            alert("已清空");
            smsSwitchTab('chat-detail');
            smsRenderChatDetail();
        }
    }
};

window.smsDeleteChat = function() {
    if(confirm("确定要删除这个会话吗？")) {
        smsState.chats = smsState.chats.filter(c => c.id !== smsState.activeChatId);
        smsState.activeChatId = null;
        smsSaveData();
        smsSwitchTab('chats');
        smsRenderList();
    }
};

// 15. 马甲切换逻辑
window.smsOpenSenderSheet = function() {
    document.getElementById('sms-sender-sheet-modal').classList.add('active');
};
window.closeSmsSenderSheet = function() {
    document.getElementById('sms-sender-sheet-modal').classList.remove('active');
};
window.smsSelectSender = function(senderName) {
    smsState.currentSender = senderName;
    localStorage.setItem('ios_sms_current_sender', senderName); // 👇 新增：持久化保存身份
    const displayEl = document.getElementById('sms-current-sender-display');
    if (displayEl) displayEl.innerText = senderName;
    closeSmsSenderSheet();
};
window.smsAddCustomSender = function() {
    if (typeof wcOpenGeneralInput === 'function') {
        wcOpenGeneralInput("请输入自定义号码或身份名称", (name) => {
            if (name && name.trim() !== "") {
                smsSelectSender(name.trim());
            }
        });
    } else {
        const name = prompt("请输入自定义号码或身份名称：");
        if (name && name.trim() !== "") {
            smsSelectSender(name.trim());
        }
    }
};
// ==========================================
// 短信长按菜单 (编辑/删除/重Roll)
// ==========================================
let smsLongPressTimer = null;
let smsSelectedMsgId = null;

window.handleSmsTouchStart = function(e, msgId) {
    smsLongPressTimer = setTimeout(() => {
        const touch = e.touches[0];
        showSmsContextMenu(touch.clientX, touch.clientY, msgId);
    }, 500);
};

window.handleSmsTouchEnd = function() {
    if (smsLongPressTimer) {
        clearTimeout(smsLongPressTimer);
        smsLongPressTimer = null;
    }
};

window.showSmsContextMenu = function(eOrX, yOrMsgId, msgIdIfTouch) {
    let x, y, msgId;
    if (typeof eOrX === 'object') {
        eOrX.preventDefault();
        eOrX.stopPropagation();
        x = eOrX.pageX || eOrX.clientX;
        y = eOrX.pageY || eOrX.clientY;
        msgId = yOrMsgId;
    } else {
        x = eOrX;
        y = yOrMsgId;
        msgId = msgIdIfTouch;
    }

    smsSelectedMsgId = msgId;
    
    let menu = document.getElementById('sms-context-menu');
    if (!menu) {
        menu = document.createElement('div');
        menu.id = 'sms-context-menu';
        menu.className = 'dream-context-menu'; // 复用梦境的横向菜单样式
        document.body.appendChild(menu);
    }
    
    const chat = smsState.chats.find(c => c.id === smsState.activeChatId);
    if (!chat) return;
    const msg = chat.history.find(m => m.id === msgId);
    const isAI = msg && msg.sender === 'them';

    let menuHtml = '';
    if (isAI) {
        menuHtml += `
            <div class="dream-ctx-item" onclick="smsActionRoll()">
                <svg viewBox="0 0 24 24"><polyline points="23 4 23 10 17 10"></polyline><path d="M20.49 15a9 9 0 1 1-2.12-9.36L23 10"></path></svg>
            </div>
        `;
    }
    menuHtml += `
        <div class="dream-ctx-item" onclick="smsActionEdit()">
            <svg viewBox="0 0 24 24"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path></svg>
        </div>
        <div class="dream-ctx-item" onclick="smsActionDelete()">
            <svg viewBox="0 0 24 24"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg>
        </div>
    `;
    menu.innerHTML = menuHtml;

    const menuWidth = isAI ? 180 : 120;
    const menuHeight = 44; 
    const screenW = window.innerWidth;
    
    let leftPos = x - (menuWidth / 2);
    let topPos = y - menuHeight - 20;

    if (leftPos < 10) leftPos = 10;
    if (leftPos + menuWidth > screenW - 10) leftPos = screenW - menuWidth - 10;

    if (topPos < 10) {
        topPos = y + 30;
        menu.style.setProperty('--triangle-top', '-7px');
        menu.style.setProperty('--triangle-bottom', 'auto');
        menu.style.setProperty('--triangle-rotate', '180deg');
    } else {
        menu.style.setProperty('--triangle-top', '100%');
        menu.style.setProperty('--triangle-bottom', 'auto');
        menu.style.setProperty('--triangle-rotate', '0deg');
    }

    menu.style.left = leftPos + 'px';
    menu.style.top = topPos + 'px';
    menu.style.display = 'flex';
};

// 全局点击隐藏菜单
document.addEventListener('click', (e) => {
    const menu = document.getElementById('sms-context-menu');
    if (menu && menu.style.display === 'flex') {
        if (!e.target.closest('#sms-context-menu') && !e.target.closest('.sms-bubble')) {
            menu.style.display = 'none';
            smsSelectedMsgId = null;
        }
    }
});

window.smsActionEdit = function() {
    const chat = smsState.chats.find(c => c.id === smsState.activeChatId);
    if (!chat) return;
    const msg = chat.history.find(m => m.id === smsSelectedMsgId);
    if (!msg) return;

    document.getElementById('sms-context-menu').style.display = 'none';

    openIosTextEditModal("编辑短信", msg.content, (newText) => {
        if (newText) {
            msg.content = newText;
            smsSaveData();
            smsRenderChatDetail();
        }
    });
};

window.smsActionDelete = function() {
    const chat = smsState.chats.find(c => c.id === smsState.activeChatId);
    if (!chat) return;

    document.getElementById('sms-context-menu').style.display = 'none';

    if (confirm("确定删除这条短信吗？")) {
        chat.history = chat.history.filter(m => m.id !== smsSelectedMsgId);
        smsSaveData();
        smsRenderChatDetail();
    }
};

window.smsActionRoll = function() {
    const chat = smsState.chats.find(c => c.id === smsState.activeChatId);
    if (!chat) return;

    document.getElementById('sms-context-menu').style.display = 'none';

    const msgIndex = chat.history.findIndex(m => m.id === smsSelectedMsgId);
    if (msgIndex > -1) {
        const isLastMsg = msgIndex === chat.history.length - 1;
        if (!isLastMsg) {
            if (!confirm("重生成此条消息，将会删除它之后的所有对话记录，确定要继续吗？")) {
                return;
            }
        }
        // 截断数组
        chat.history = chat.history.slice(0, msgIndex);
        smsSaveData();
        smsRenderChatDetail();
        
        // 重新触发 AI
        smsTriggerAI_Manual();
    }
};

// 数据持久化
async function smsSaveData() {
    await idb.set('ios_sms_data', smsState.chats);
}
async function smsLoadData() {
    const data = await idb.get('ios_sms_data');
    if (data) smsState.chats = data;
}

// 初始化
window.addEventListener('load', async () => {
    await smsLoadData();
});
// ==========================================
// 新版全屏面具 (Masks) 渲染逻辑
// ==========================================
window.wcRenderFullScreenMasks = function() {
    const list = document.getElementById('wc-masks-fullscreen-list');
    if (!list) return;
    list.innerHTML = '';

    if (!wcState.masks || wcState.masks.length === 0) {
        list.innerHTML = '<div style="text-align:center; color:#999; margin-top:50px; font-size:14px;">暂无面具，点击右上角 ＋ 添加吧</div>';
        return;
    }

    wcState.masks.forEach(mask => {
        const isCurrent = wcState.user.maskId === mask.id || (wcState.user.name === mask.name && wcState.user.avatar === mask.avatar);
        const currentTag = isCurrent ? '<span style="color:#007AFF; font-size:10px; margin-left:4px;">(当前)</span>' : '';

        const card = document.createElement('div');
        card.className = 'mask-card';
        
        card.innerHTML = `
            <div class="mask-card-tabs">
                <div class="mask-tab-main" onclick="wcApplyMask(${mask.id})" title="点击应用此面具">
                    ${mask.name} ${currentTag}
                </div>
                <div class="mask-tab-edit" onclick="wcOpenEditMask(${mask.id})">Edit</div>
                <div class="mask-tab-delete" onclick="wcDeleteMask(${mask.id})">Delete</div>
            </div>
            <div class="mask-card-body">
                <div class="mask-card-left">
                    <div class="mask-avatar-wrapper">
                        <svg class="mask-star star-tl" viewBox="0 0 24 24"><path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/></svg>
                        <svg class="mask-star star-tr" viewBox="0 0 24 24"><path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/></svg>
                        <svg class="mask-star star-bl" viewBox="0 0 24 24"><path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/></svg>
                        <svg class="mask-star star-br" viewBox="0 0 24 24"><path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/></svg>
                        <img src="${mask.avatar}" class="mask-avatar-img">
                    </div>
                    <div class="mask-char-name">${mask.name}</div>
                </div>
                <div class="mask-card-right">
                    <div class="mask-gender-box">
                        <span class="mask-gender-label">GENDER</span>
                        <span class="mask-gender-value">${mask.gender || '保密'}</span>
                    </div>
                    <div class="mask-prompt-box">
                        <div class="mask-prompt-label">
                            <svg viewBox="0 0 24 24"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path><circle cx="12" cy="7" r="4"></circle></svg>
                            PROMPT
                        </div>
                        <div class="mask-prompt-text">
                            ${mask.prompt || '暂无设定...'}
                        </div>
                    </div>
                </div>
            </div>
        `;
        list.appendChild(card);
    });
};
