# Arquivo_Telegram

-  Extraindo dados de um banco atraves do Mysql direcionando a saida para um arquivo.csv e encaminhando para um chat do telegran atraves da API ===
- Para executar os scripts, deverão estar na mesma pasta ===


# Arquivo 1  
==Arquivo Client==

[client]

user="seu_usuario_de_banco"

password="Sua_senha_de_banco"

database="Seu_database"

# Arquivo 2 
== Script para executar no inicio do mes, buscando dados do mes anterior e direcionando a saida para um arquivo.csv ==

 #!/bin/bash

database="Seu_database"

#Obtendo a data atual 
current_date=$(date +"%Y-%m-%d")

current_month=$(date +"%Y-%m-01")

#Calculando o primeiro dia e o último dia do mês atual
start_date="${current_month} 00:00:00"

end_date=$(date -d "$current_month +1 month -1 second" +"%Y-%m-%d %H:%M:%S")

#Criando um nome de arquivo com a data do mês
output_filename="./Pesquisa/Pesquisa_$(date +"%Y-%m").csv"

#Executando o comando MySQL e redirecione a saída para o arquivo:
mysql --defaults-file=mysql_options.cnf -D "$database" -e "SELECT id, data, id_pesquisa, id_service, id_resposta, nota_resposta, CONCAT('Ramal ', operador) AS operador FROM pesquisa WHERE data BETWEEN 
'$start_date' AND '$end_date';" > "$output_filename"

# Arquivo 3 = Este Script lê a API do telegram converte para limguagem de maquina e envia para o telegram:


#!/usr/bin/python
-*- coding: utf-8 -*-
import telegram
from telegram import InputFile
import asyncio
import os
import subprocess
import glob

#Substitua 'TOKEN' pelo token de API do seu bot
TOKEN = 'Seu_Token'

#Diretório onde os arquivos de pesquisa são armazenados
output_directory = './Pesquisa/'

#chat ID do Telegram:
chat_id = 'Seu_Chat_ID'

#Executando primeiramente o bash script
subprocess.run(["./script_banco_pesquisa.sh"])

#Aguardando até que o primeiro script seja concluído e o arquivo seja gerado:
while not glob.glob(os.path.join(output_directory, 'Pesquisa_*.csv')):
    asyncio.sleep(1)

#Encontre o arquivo mais recente no diretório
latest_file = max(glob.glob(os.path.join(output_directory, 'Pesquisa_*.csv')), key=os.path.getctime)

#Crie uma instância do bot
bot = telegram.Bot(token=TOKEN)

async def send_document():
    with open(latest_file, 'rb') as file:
        # Use o nome do arquivo mais recente como o nome do arquivo no Telegram
        filename = os.path.basename(latest_file)
        await bot.send_document(chat_id=chat_id, document=InputFile(file, filename=filename))

async def main():
    await send_document()

if __name__ == "__main__":
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main())


