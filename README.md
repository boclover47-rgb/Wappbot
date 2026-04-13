Whatsappbot/
const { default: makeWASocket, useMultiFileAuthState } = require("@whiskeysockets/baileys");

let parole = {
    parola1: 0,
    parola2: 0,
    parola3: 0
};

// 👤 contatore utenti
let utenti = {};

function aggiornaUtente(nome) {
    if (!utenti[nome]) {
        utenti[nome] = 0;
    }
    utenti[nome]++;
}

function getClassifica() {
    let lista = Object.entries(utenti);

    if (lista.length === 0) {
        return "Nessun dato ancora 😄";
    }

    lista.sort((a, b) => b[1] - a[1]);

    let testo = "🏆 CLASSIFICA:\n\n";

    lista.forEach((u, i) => {
        testo += `${i + 1}. ${u[0]} — ${u[1]} punti\n`;
    });

    return testo;
}

async function start() {
    const { state, saveCreds } = await useMultiFileAuthState("auth");

    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true
    });

    sock.ev.on("creds.update", saveCreds);

    sock.ev.on("messages.upsert", async ({ messages }) => {
        const msg = messages[0];
        if (!msg.message) return;

        const testo =
            msg.message.conversation ||
            msg.message.extendedTextMessage?.text;

        if (!testo) return;

        const lower = testo.toLowerCase();
        const sender = msg.pushName || "anonimo";

        // 🏆 comando !cl
        if (lower === "!cl") {
            await sock.sendMessage(msg.key.remoteJid, {
                text: getClassifica()
            });
            return;
        }

        // 📊 controllo parole
        for (let p in parole) {
            if (lower.includes(p)) {
                parole[p]++;
                aggiornaUtente(sender);

                await sock.sendMessage(msg.key.remoteJid, {
                    text: `📊 "${p}" contata!\nTotale: ${parole[p]}`
                });
                break;
            }
        }
    });
}

start();
