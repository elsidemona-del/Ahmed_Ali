const { Client, LocalAuth, MessageMedia } = require('whatsapp-web.js');
const { GoogleGenAI } = require('@google/genai');
const fs = require('fs');
const path = require('path');

const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

const folders = ['gym_videos', 'outfit_videos', 'voice_notes'];
for (const folder of folders) {
  const dirPath = path.join(__dirname, folder);
  if (!fs.existsSync(dirPath)) {
    fs.mkdirSync(dirPath, { recursive: true });
  }
}

function getRandomMedia(folderName) {
  const dirPath = path.join(__dirname, folderName);
  if (!fs.existsSync(dirPath)) return null;
  const files = fs
    .readdirSync(dirPath)
    .filter((f) => !f.startsWith('.') && fs.statSync(path.join(dirPath, f)).isFile());
  if (files.length === 0) return null;
  const chosen = files[Math.floor(Math.random() * files.length)];
  return MessageMedia.fromFilePath(path.join(dirPath, chosen));
}

function cleanSingleLine(text) {
  return (text || '')
    .replace(/[\r\n]+/g, ' ')
    .replace(/[.,،؛:!?؟…]+/g, ' ')
    .replace(/\s+/g, ' ')
    .trim();
}

const SYSTEM_PROMPT = `أنت شاب مصري اسمك أحمد.
شخصيتك حنين جدا وبتحب بصدق، وبتغير وبتتعصب بحب ودلع، وبتخاف عليها.
عيلتك فيها أختك آلاء وأخوك محمد وبتذكرهم بشكل طبيعي لما يناسب الكلام.
قواعد الرد الصارمة:
1. رد دايما بجملة واحدة قصيرة فقط.
2. ممنوع تماما استخدام الفواصل أو النقط أو علامات الترقيم.
3. كلامك مصري طبيعي ودافئ وخفيف.`;

async function generateAhmedReply(userText) {
  try {
    const response = await ai.models.generateContent({
      model: 'gemini-2.5-flash',
      contents: userText,
      config: {
        systemInstruction: SYSTEM_PROMPT,
      },
    });
    const cleaned = cleanSingleLine(response.text);
    return cleaned || 'وحشتيني يا روحي قلبي معاك دايما';
  } catch (err) {
    return 'معلش الشبكة رخمة شوية بس قلبي معاك';
  }
}

const client = new Client({
  authStrategy: new LocalAuth(),
  puppeteer: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox'],
  },
});

let lastActiveChatId = null;

client.on('qr', (qr) => {
  console.log('QR_CODE:', qr);
});

client.on('ready', () => {
  console.log('Ahmed WhatsApp bot is ready!');

  setInterval(async () => {
    if (!lastActiveChatId) return;
    const roll = Math.random();
    if (roll < 0.35) {
      const msg = await generateAhmedReply('ابعتلها رسالة تلقائية من نفسك تعبر فيها عن حبك أو غيرتك أو تطمن عليها');
      await client.sendMessage(lastActiveChatId, msg);
    } else if (roll < 0.6) {
      const targetFolder = Math.random() < 0.5 ? 'gym_videos' : 'outfit_videos';
      const media = getRandomMedia(targetFolder);
      if (media) {
        const cap = await generateAhmedReply('ابعتلها فيديو عشوائي وعلق بجملة واحدة قصيرة');
        await client.sendMessage(lastActiveChatId, media, { caption: cap });
      }
    } else if (roll < 0.8) {
      const voice = getRandomMedia('voice_notes');
      if (voice) {
        await client.sendMessage(lastActiveChatId, voice, { sendAudioAsVoice: true });
      }
    }
  }, 1000 * 60 * 45);
});

client.on('message', async (msg) => {
  if (msg.fromMe) return;
  lastActiveChatId = msg.from;
  const body = (msg.body || '').trim();
  const lower = body.toLowerCase();

  try {
    const reactions = ['❤️', '😍', '🔥', '😂', '🥺'];
    await msg.react(reactions[Math.floor(Math.random() * reactions.length)]);
  } catch (e) {}

  if (lower.includes('اوتفيت') || lower.includes('أوتفيت') || lower.includes('outfit')) {
    const media = getRandomMedia('outfit_videos');
    if (media) {
      const cap = await generateAhmedReply('هي طلبت تشوف الأوتفيت بتاعك');
      await client.sendMessage(msg.from, media, { caption: cap });
      return;
    }
  }

  if (lower.includes('جيم') || lower.includes('gym') || lower.includes('تمرين')) {
    const media = getRandomMedia('gym_videos');
    if (media) {
      const cap = await generateAhmedReply('هي جابت سيرة الجيم والتمرين');
      await client.sendMessage(msg.from, media, { caption: cap });
      return;
    }
  }

  if (lower.includes('صوت') || lower.includes('فويس') || lower.includes('voice')) {
    const voice = getRandomMedia('voice_notes');
    if (voice) {
      await client.sendMessage(msg.from, voice, { sendAudioAsVoice: true });
      return;
    }
  }

  const reply = await generateAhmedReply(body || 'بعتتلك رسالة');
  await client.sendMessage(msg.from, reply);
});

client.initialize();
