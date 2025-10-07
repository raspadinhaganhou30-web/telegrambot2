import telegram
import schedule
import time
import random
import logging
from threading import Thread
from datetime import datetime

# ================= CONFIGURAÇÃO =================
TOKEN = "7337195248:AAFyB2QAReKuaU_49A--LKP184XEkBtjdxI"
CHAT_ID = "-1003198115663"
TOPIC_ID = 7

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(message)s')
bot = telegram.Bot(token=TOKEN)

# ================= BANCO DE DADOS =================
DOMINIOS = {
    "🥇 @sortuda.com": 200,
    "💎 @vip.com": 250,
    "🚀 @ganhei.com": 300,
    "👑 @sorte.com": 350
}

HORARIOS_PICO = {
    "08h-09h": "Fluxo matinal forte - horário estratégico!",
    "09h-10h": "Pico de acessos - maior liquidez!",
    "10h-11h": "Momento de alta volatilidade - grandes oportunidades!",
    "11h-12h": "Pré-almoço com retornos consistentes!",
    "12h-13h": "Horário de almoço - movimento intenso!",
    "13h-14h": "Pós-almoço com tendência de alta!",
    "14h-15h": "Início da tarde com momentum positivo!",
    "15h-16h": "Virada europeia - oportunidades únicas!",
    "16h-17h": "Pico da tarde - melhores retornos!",
    "17h-18h": "Final do dia útil - última chance diurna!",
    "18h-19h": "Início da noite - volatilidade aumentando!",
    "19h-20h": "Janta dos ganhadores - horário premium!",
    "20h-21h": "Noite com alta probabilidade!",
    "21h-22h": "Pico noturno - maiores bônus!",
    "22h-23h": "Madrugada chegando - oportunidades ousadas!",
    "23h-00h": "Última hora - tudo ou nada!"
}

DICAS = [
    "Sistema identificou padrão positivo - entrada recomendada!",
    "Alta probabilidade de retornos acima da média!",
    "Momento estratégico com baixa concorrência!",
    "Janela de oportunidade limitada - ação imediata!",
    "Fluxo consistente identificado - timing perfeito!",
    "Condições ideais para multiplicação rápida!",
    "Sinal técnico forte - confiança elevada!",
    "Oportunidade exclusiva deste horário!"
]

# ================= GERADOR DE SINAIS =================
def gerar_sinal_horario():
    try:
        hora_atual = datetime.now().strftime("%Hh-%Hh")
        periodo = list(HORARIOS_PICO.keys())[list(HORARIOS_PICO.keys()).index(hora_atual) if hora_atual in HORARIOS_PICO else 0]
        
        dominio, bonus = random.choice(list(DOMINIOS.items()))
        confianca = random.randint(80, 95)
        dica = random.choice(DICAS)
        descricao_horario = HORARIOS_PICO[periodo]
        
        # Ajusta bônus conforme horário
        if "21h-22h" in periodo or "22h-23h" in periodo:
            bonus += 50  # Bônus extra na madrugada
        
        sinal = f"""🕐 SINAL {periodo.replace('h-', ':00-').replace('h', ':00')}

🔥 DOMÍNIO: {dominio}
💰 BÔNUS: {bonus}%
🎯 CONFIANÇA: {confianca}% ✅

📊 {descricao_horario}
💡 {dica}

🔄 PRÓXIMO SINAL: Em 1 hora ⏳

🎪 Acesse: https://raspadinhawin.org/"""
        
        return sinal
        
    except Exception as e:
        logging.error(f"Erro ao gerar sinal: {e}")
        return None

# ================= ENVIO DE SINAIS =================
def enviar_sinal_horario():
    try:
        sinal = gerar_sinal_horario()
        if sinal:
            # Teclado inline com botão
            keyboard = {
                'inline_keyboard': [[
                    {'text': '🎯 ACESSAR AGORA', 'url': 'https://raspadinhawin.org/'},
                    {'text': '📊 VER ESTATÍSTICAS', 'url': 'https://raspadinhawin.org/stats'}
                ]]
            }
            
            bot.send_message(
                chat_id=CHAT_ID,
                text=sinal,
                message_thread_id=TOPIC_ID,
                reply_markup=keyboard
            )
            logging.info(f"Sinal horário enviado com sucesso!")
            
            # Log de performance
            hora = datetime.now().strftime("%H:%M")
            logging.info(f"✅ Sinal das {hora} distribuído!")
            
        else:
            logging.error("Falha ao gerar sinal horário")
            
    except Exception as e:
        logging.error(f"Erro ao enviar sinal horário: {e}")

# ================= AGENDADOR HORÁRIO =================
def agendar_sinais_horarios():
    # Agenda a cada hora nos minutos 00
    for hora in range(24):
        schedule.every().day.at(f"{hora:02d}:00").do(enviar_sinal_horario)
    
    # Agendamentos extras para picos
    schedule.every().day.at("08:30").do(enviar_sinal_horario)
    schedule.every().day.at("12:30").do(enviar_sinal_horario)
    schedule.every().day.at("18:30").do(enviar_sinal_horario)
    schedule.every().day.at("21:30").do(enviar_sinal_horario)
    
    logging.info("🤖 Agendador horário iniciado - 24/7!")
    
    while True:
        schedule.run_pending()
        time.sleep(30)  # Verifica a cada 30 segundos

# ================= SISTEMA DE HEALTH CHECK =================
def health_check():
    while True:
        try:
            bot.get_me()
            # Log a cada 6 horas
            if datetime.now().hour % 6 == 0:
                logging.info("❤️ Bot saudável e rodando...")
            time.sleep(3600)  # 1 hora
        except Exception as e:
            logging.error(f"❌ Health check falhou: {e}")
            time.sleep(300)  # Espera 5 minutos e tenta novamente

# ================= COMANDOS DO BOT =================
def processar_comandos():
    last_update_id = None
    
    while True:
        try:
            updates = bot.get_updates(offset=last_update_id, timeout=10)
            
            for update in updates:
                if update.update_id > (last_update_id or 0):
                    last_update_id = update.update_id + 1
                    
                    if update.message and update.message.text:
                        comando = update.message.text
                        chat_id = update.message.chat_id
                        
                        if comando == "/sinal":
                            sinal = gerar_sinal_horario()
                            if sinal:
                                bot.send_message(chat_id=chat_id, text=sinal)
                                
                        elif comando == "/status":
                            bot.send_message(
                                chat_id=chat_id, 
                                text="🤖 Bot operacional 24/7\n✅ Sinais horários ativos\n🎯 Enviando a cada 60min"
                            )
                            
                        elif comando == "/ajuda":
                            ajuda_texto = """
🎯 COMANDOS DISPONÍVEIS:

/sinal - Receba o sinal atual
/status - Status do bot
/ajuda - Esta mensagem

📢 Sinais automáticos a cada 1 hora!
                            """
                            bot.send_message(chat_id=chat_id, text=ajuda_texto)
                            
        except Exception as e:
            logging.error(f"Erro nos comandos: {e}")
            time.sleep(10)

# ================= INICIALIZAÇÃO =================
if __name__ == "__main__":
    logging.info("🚀 INICIANDO BOT DE SINAIS HORÁRIOS...")
    
    # Iniciar threads
    Thread(target=agendar_sinais_horarios, daemon=True).start()
    Thread(target=health_check, daemon=True).start()
    Thread(target=processar_comandos, daemon=True).start()
    
    # Manter principal ativo
    try:
        logging.info("✅ Bot totalmente operacional!")
        while True:
            time.sleep(60)
    except KeyboardInterrupt:
        logging.info("⏹️ Bot encerrado pelo usuário")
