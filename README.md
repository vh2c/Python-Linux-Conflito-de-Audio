# Python Linux Conflito de Audio - Python sem audio
Existe um conflito de audio quando rodamos executáveis em PYTHON no Linux, impossibilitando rodar arquivos wav ou mp3 através de scripts em Python.
Esse conflito acontece porque o Linux utiliza o PulseAudio para gerenciamento do ALSA, bibliotecas de audio e hardwares.
O PulseAudio é concebido para rodar em ambiente de usuário, já o Python entra no Linux através das bibliotecas os.system, subprocess, pygame, pyaudio, simpleaudio, etc, etc,etc como ROOT...portanto sem usuário - Uid = 0.
Essa combinação faz com que seu script funcione isoladamente, porém assim que um script "disputa" a atenção do PulseAudio, seu script fica mudo, gerando erros do tipo "ERROR OPENING PCM | ERROR DEVICE | RESOURCE BUSY".

## Solução
Rodar o AudioPulse em modo WIDE | --system

## TUTORIAL

1. Criar arquivo /etc/systemd/system/pulseaudio.service

conteudo:
```
    [Unit]
    Description=PulseAudio System Server

    [Service]
    Type=notify
    ExecStart=/bin/pulseaudio --daemonize=no --system --realtime --log-target=journal

    [Install]
    WantedBy=multi-user.target
```


2. Desabilitar PulseAudio USER

    ```sudo systemctl --global disable pulseaudio.service pulseaudio.socket```


3. Atualizar /etc/pulse/client.conf

    adicionar:
    
        default-server = /var/run/pulse/native


4. Adicionar usuarios ao grupo pulse-audio

    ```sudo usermod -a -G pulse-audio USUARIO```


5. Habilitar serviço automaticamente

    ```sudo systemctl --system enable pulseaudio.service```


6. Iniciar serviço sem Boot

    ```sudo systemctl --system start pulseaudio.service```


7. Verificar se está rodando:

    ```sudo systemctl --system status pulseaudio.service```


8. ENCONTRAR MANUALMENTE A CONFIGURACAO DE SAIDA, CASO A SUA NAO ESTEJA FUNCIONANDO

    a.Escolher um arquivo .wav e substituir o caminho /usr/share...
    
    b.Ir substituindo os números 0,1 por 0,2 | 0,3 | 1,0 | 1,1 etc até encontrar o hardware que funcione 

    ```aplay -D plughw:0,1 /usr/share/sounds/alsa/Front_Right.wav```


9. Atualizar /etc/pulse/default.pa

    adicionar:
```
        load-module module-alsa-sink device=hw:1,7
```
   Onde 1,7 são os números que vc encontrou na etapa 8 e que funcionaram com o audio

10. REBOOT

11. PROBLEMA -> A configuração do audio funciona quando eu rodo a etapa 9 no terminal, mas o Pulseaudio não carregou o load-module do arquivo default.pa:

   Ir para "Aplicativos de Inicialização" e "Adicionar Comando Personalizado":
       NOME: Carregar Audio
       COMANDO: pactl load-module module-alsa-sink device=hw:1,7
       >> Não se esqueça os número 1,7 são os mesmos que vc verificou que funcionaram no seu hardware
       COMENTARIO: Carregar Hardware no PulseAudio 

12. Novo REBOOT

===============================================================================
###### Referencia: https://wiki.archlinux.org/index.php/PulseAudio/Examples
===============================================================================
