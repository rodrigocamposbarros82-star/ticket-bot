[package.json](https://github.com/user-attachments/files/24240738/package.json)

[index.js](https://github.com/user-attachments/files/24256568/index.js)
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
    { label: 'Théo Tuguinha 744' , value: '1202319235334799433' },
    { label: 'Abel Tuguinha 1032 ' , value: '1328022884244783238' }
];

const CANAL_AVALIACAO_ID = '1451237259591286846'; 
const CANAL_DIRETORIA_ID = '1451236685642993766';
const CANAL_LOG_FUNCIONARIOS_ID = '1451236651408953464';

client.once('ready', async () => {
    console.log(`✅ Bot online como ${client.user.tag}`);

    const canal = client.channels.cache.get(CANAL_AVALIACAO_ID);
    if (canal) {
        await canal.bulkDelete(5).catch(() => null);

        const menu = new StringSelectMenuBuilder()
            .setCustomId('selecionar_funcionario')
            .setPlaceholder('Selecione o profissional que te atendeu')
            .addOptions(funcionarios);

        const row = new ActionRowBuilder().addComponents(menu);

        const embed = new EmbedBuilder()
            .setTitle('🏥 Hospital Cidade Nova - Avaliação')
            .setDescription('Sua opinião é fundamental! Selecione o profissional abaixo no menu para avaliar o atendimento.')
            .setColor('#0099ff')
            .setFooter({ text: 'Sistema de Gestão Hospitalar' });

        await canal.send({ embeds: [embed], components: [row] });
    }
});

client.on(Events.InteractionCreate, async interaction => {
    if (interaction.isStringSelectMenu() && interaction.customId === 'selecionar_funcionario') {
        const funcionarioID = interaction.values[0];
        const estrelas = new ActionRowBuilder();
        
        // REMOVIDO O ASTERISCO E ADICIONADO A ESTRELA ⭐
        for (let i = 1; i <= 5; i++) {
            estrelas.addComponents(
                new ButtonBuilder()
                    .setCustomId(`estrela_${i}_${funcionarioID}`)
                    .setLabel(`${i} ⭐`) 
                    .setStyle(ButtonStyle.Primary)
            );
        }
        await interaction.reply({ content: `🩺 Avaliando profissional: <@${funcionarioID}>`, components: [estrelas], ephemeral: true });
    }

    if (interaction.isButton() && interaction.customId.startsWith('estrela_')) {
        const [_, nota, funcID] = interaction.customId.split('_');
        const modal = new ModalBuilder()
            .setCustomId(`modal_${nota}_${funcID}`)
            .setTitle('Feedback do Atendimento');

        const input = new TextInputBuilder()
            .setCustomId('comentario')
            .setLabel('Deixe seu comentário (opcional):')
            .setStyle(TextInputStyle.Paragraph)
            .setRequired(false);

        modal.addComponents(new ActionRowBuilder().addComponents(input));
        await interaction.showModal(modal);
    }

    if (interaction.isModalSubmit() && interaction.customId.startsWith('modal_')) {
        const [_, nota, funcID] = interaction.customId.split('_');
        const comentario = interaction.fields.getTextInputValue('comentario') || 'Sem comentário.';
        
        const embedFeedback = new EmbedBuilder()
            .setTitle('📋 Registro de Avaliação')
            .addFields(
                { name: '👨‍⚕️ Profissional', value: `<@${funcID}>`, inline: true },
                { name: '⭐ Nota', value: `${nota}/5`, inline: true },
                { name: '👤 Paciente', value: `${interaction.user.tag}`, inline: false },
                { name: '💬 Comentário', value: comentario }
            )
            .setColor(nota >= 4 ? '#2ecc71' : '#e74c3c')
            .setTimestamp();

        const canalDiretoria = client.channels.cache.get(CANAL_DIRETORIA_ID);
        if (canalDiretoria) await canalDiretoria.send({ embeds: [embedFeedback] });

        const canalPublico = client.channels.cache.get(CANAL_LOG_FUNCIONARIOS_ID);
        if (canalPublico) await canalPublico.send({ embeds: [embedFeedback] });

        await interaction.reply({ content: '✅ Sua avaliação foi registrada com sucesso!', ephemeral: true });
    }
});

client.login(process.env.TOKEN);

// --- SERVIDOR WEB NO FINAL PARA REPLIT 24/7 ---
import express from 'express';
const app = express();

app.get('/', (req, res) => {
  res.send('O Bot do Hospital esta online!');
});

app.listen(3000, () => {
  console.log('🌐 Servidor de monitoramento rodando na porta 3000');
});
