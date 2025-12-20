[package.json](https://github.com/user-attachments/files/24273919/package.json)[Uploading pac{
  "name": "hospital-bot",
  "version": "1.0.0",
  "description": "Bot de avaliação Hospital Cidade Nova",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "discord.js": "^14.25.1",
    "dotenv": "^16.6.1"
  }
}
kage.json…]()

[index.js](https://github.com/user-attachments/files/24273921/index.js)
import 'dotenv/config';
import {
    Client,
    GatewayIntentBits,
    ActionRowBuilder,
    ButtonBuilder,
    ButtonStyle,
    StringSelectMenuBuilder,
    ModalBuilder,
    TextInputBuilder,
    TextInputStyle,
    Events,
    EmbedBuilder
} from 'discord.js';

import express from 'express';

const client = new Client({
    intents: [
        GatewayIntentBits.Guilds,
        GatewayIntentBits.GuildMessages,
        GatewayIntentBits.MessageContent,
        GatewayIntentBits.GuildMembers
    ]
});

const funcionarios = [
    { label: 'Alcatra Soares 727', value: '970489711980326962' },
    { label: 'ARCANJO RENEGADO 1251', value: '1429572810707177616' },
    { label: 'Don Couto 704', value: '1095798501922832487' },
    { label: 'Japa Scott 1344', value: '1368185415726665758' },
    { label: 'Kleber 1273', value: '534397463478665236' },
    { label: 'Lauren 869', value: '1328551804031467563' },
    { label: 'Marechal WINCHESTER 641', value: '1253369256339312781' },
    { label: 'Mateus 1041', value: '1390612126527262811' },
    { label: 'Scotin Cria 1004', value: '1282370819149926535' },
    { label: 'THURZINN MASCOTE 1370', value: '1298444688239362118' },
    { label: 'Théo Tuguinha 744', value: '1202319235334799433' },
    { label: 'Abel Tuguinha 1032', value: '1328022884244783238' },
    { label: 'Miguel Horacio 1115', value: '966033281420251178' },
    { label: 'Samuel Horacio 915', value: '964358435053387836' }
];

const CANAL_AVALIACAO_ID = '1451237259591286846';
const CANAL_DIRETORIA_ID = '1451236685642993766';
const CANAL_LOG_FUNCIONARIOS_ID = '1451236651408953464';

/* ================= FUNÇÃO DE REGISTRO ================= */
async function registrarAvaliacao(interaction, nota, funcID, comentario) {
    const embedFeedback = new EmbedBuilder()
        .setTitle('📋 Registro de Avaliação')
        .addFields(
            { name: '👨‍⚕️ Profissional', value: `<@${funcID}>`, inline: true },
            { name: '⭐ Nota', value: `${nota}/5`, inline: true },
            { name: '👤 Paciente', value: interaction.user.tag, inline: false },
            { name: '💬 Comentário', value: comentario }
        )
        .setColor(nota >= 4 ? '#2ecc71' : '#e74c3c')
        .setTimestamp();

    const diretoria = interaction.client.channels.cache.get(CANAL_DIRETORIA_ID);
    if (diretoria) await diretoria.send({ embeds: [embedFeedback] });

    const logs = interaction.client.channels.cache.get(CANAL_LOG_FUNCIONARIOS_ID);
    if (logs) await logs.send({ embeds: [embedFeedback] });

    await interaction.reply({
        content: '✅ Sua avaliação foi registrada com sucesso!',
        ephemeral: true
    });
}

/* ================= BOT ONLINE ================= */
client.once('ready', async () => {
    console.log(`✅ Bot online como ${client.user.tag}`);

    const canal = client.channels.cache.get(CANAL_AVALIACAO_ID);
    if (!canal) return;

    await canal.bulkDelete(5).catch(() => null);

    const menu = new StringSelectMenuBuilder()
        .setCustomId('selecionar_funcionario')
        .setPlaceholder('Selecione o profissional que te atendeu')
        .addOptions(funcionarios);

    const row = new ActionRowBuilder().addComponents(menu);

    const embed = new EmbedBuilder()
        .setTitle('🏥 Hospital Cidade Nova - Avaliação')
        .setDescription(
            'Sua opinião é fundamental!\nSelecione o profissional abaixo para avaliar o atendimento.'
        )
        .setColor('#0099ff')
        .setFooter({ text: 'Sistema de Gestão Hospitalar' });

    await canal.send({ embeds: [embed], components: [row] });
});

/* ================= INTERAÇÕES ================= */
client.on(Events.InteractionCreate, async interaction => {

    /* MENU DE FUNCIONÁRIOS */
    if (interaction.isStringSelectMenu() && interaction.customId === 'selecionar_funcionario') {
        const funcID = interaction.values[0];

        const estrelas = new ActionRowBuilder();
        for (let i = 1; i <= 5; i++) {
            estrelas.addComponents(
                new ButtonBuilder()
                    .setCustomId(`estrela_${i}_${funcID}`)
                    .setLabel(`${i} ⭐`)
                    .setStyle(ButtonStyle.Primary)
            );
        }

        await interaction.reply({
            content: `🩺 Avaliando profissional: <@${funcID}>`,
            components: [estrelas],
            ephemeral: true
        });
    }

    /* CLICOU NA ESTRELA */
    if (interaction.isButton() && interaction.customId.startsWith('estrela_')) {
        const [, nota, funcID] = interaction.customId.split('_');

        const row = new ActionRowBuilder().addComponents(
            new ButtonBuilder()
                .setCustomId(`comentar_${nota}_${funcID}`)
                .setLabel('📝 Dar comentário')
                .setStyle(ButtonStyle.Primary),
            new ButtonBuilder()
                .setCustomId(`semcoment_${nota}_${funcID}`)
                .setLabel('🚫 Não dar comentário')
                .setStyle(ButtonStyle.Secondary)
        );

        await interaction.reply({
            content: `⭐ Nota selecionada: **${nota}/5**\nDeseja deixar um comentário?`,
            components: [row],
            ephemeral: true
        });
    }

    /* DAR COMENTÁRIO */
    if (interaction.isButton() && interaction.customId.startsWith('comentar_')) {
        const [, nota, funcID] = interaction.customId.split('_');

        const modal = new ModalBuilder()
            .setCustomId(`modal_${nota}_${funcID}`)
            .setTitle('Feedback do Atendimento');

        const input = new TextInputBuilder()
            .setCustomId('comentario')
            .setLabel('Deixe seu comentário:')
            .setStyle(TextInputStyle.Paragraph)
            .setRequired(true);

        modal.addComponents(new ActionRowBuilder().addComponents(input));
        await interaction.showModal(modal);
    }

    /* NÃO DAR COMENTÁRIO */
    if (interaction.isButton() && interaction.customId.startsWith('semcoment_')) {
        const [, nota, funcID] = interaction.customId.split('_');
        await registrarAvaliacao(interaction, nota, funcID, 'Sem comentário.');
    }

    /* ENVIO DO MODAL */
    if (interaction.isModalSubmit() && interaction.customId.startsWith('modal_')) {
        const [, nota, funcID] = interaction.customId.split('_');
        const comentario = interaction.fields.getTextInputValue('comentario');

        await registrarAvaliacao(interaction, nota, funcID, comentario);
    }
});

client.login(process.env.TOKEN);

/* ================= EXPRESS 24/7 ================= */
const app = express();

app.get('/', (req, res) => {
    res.send('🤖 Bot do Hospital está online!');
});

app.listen(3000, () => {
    console.log('🌐 Servidor web rodando na porta 3000');
});
