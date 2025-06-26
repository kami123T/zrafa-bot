const { default: makeWASocket, useMultiFileAuthState, DisconnectReason, fetchLatestBaileysVersion } = require("@adiwajshing/baileys");
const P = require('pino');

async function startBot() {
    const { state, saveCreds } = await useMultiFileAuthState('./auth_info');
    const { version, isLatest } = await fetchLatestBaileysVersion();

    const sock = makeWASocket({
        version,
        auth: state,
        printQRInTerminal: true,
        logger: P({ level: 'silent' })
    });

    sock.ev.on('creds.update', saveCreds);

    sock.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect } = update;
        if(connection === 'close') {
            const shouldReconnect = (lastDisconnect.error)?.output?.statusCode !== DisconnectReason.loggedOut;
            console.log('connection closed due to ', lastDisconnect.error, ', reconnecting ', shouldReconnect);
            if(shouldReconnect) {
                startBot();
            }
        } else if(connection === 'open') {
            console.log('Bot connected');
        }
    });

    sock.ev.on('messages.upsert', async (m) => {
        // تفعيل استقبال الرسائل
        if(!m.messages) return;
        const msg = m.messages[0];
        if(!msg.message) return;
        
        const sender = msg.key.remoteJid;
        const messageType = Object.keys(msg.message)[0];

        // رد بسيط على أي رسالة نصية ترسل للبوت
        if(messageType === 'conversation') {
            const text = msg.message.conversation;
            console.log(Received message: ${text} from ${sender});

            if(text.toLowerCase() === 'مرحبا') {
                await sock.sendMessage(sender, { text: 'أهلاً! كيف أقدر أساعدك؟' });
            } else if(text.toLowerCase() === 'زرف') {
                // هنا يمكنك تضيف أمر الطرد أو أي وظيفة أخرى
                await sock.sendMessage(sender, { text: 'تم تنفيذ أمر الزرف (الطرد) - الوظيفة تحت التطوير.' });
            } else {
                await sock.sendMessage(sender, { text: 'أنا بوت بسيط الآن، اكتب "مرحبا" للتحية.' });
            }
        }
    });
}

startBot();
بوت زرافة للواتساب - فعاليات وطر
