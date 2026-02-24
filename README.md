I made a modification to the app; now it pulls an ffmpeg recording script as soon as a notification is detected.

Remember that there needs to be a script in $HOME/.local/bin/gravar-15s.sh that looks exactly like this.

## my script

```
#!/bin/bash

OUTPUT_DIR="$HOME/Vídeos/Conquistas"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
mkdir -p "$OUTPUT_DIR"

# Detecta monitor principal
MONITOR_INFO=$(xrandr | grep ' connected' | head -1)
RESOLUTION=$(echo "$MONITOR_INFO" | grep -oP '\d+x\d+\+\d+\+\d+' | head -1)

if [ -n "$RESOLUTION" ]; then
    WIDTH=$(echo "$RESOLUTION" | cut -d'x' -f1)
    HEIGHT=$(echo "$RESOLUTION" | cut -d'x' -f2 | cut -d'+' -f1)
    OFFSET_X=$(echo "$RESOLUTION" | cut -d'+' -f2)
    OFFSET_Y=$(echo "$RESOLUTION" | cut -d'+' -f3)
    OFFSET="+${OFFSET_X},${OFFSET_Y}"
else
    WIDTH=$(echo "$MONITOR_INFO" | grep -oP '\d+x\d+' | head -1 | cut -d'x' -f1)
    HEIGHT=$(echo "$MONITOR_INFO" | grep -oP '\d+x\d+' | head -1 | cut -d'x' -f2)
    OFFSET="+0,0"
fi

echo "🎮 Gravando conquista..."
echo "📺 Tela: ${WIDTH}x${HEIGHT} ${OFFSET}"
echo "⏱️  Duração: 15 segundos"

# Detecta a fonte de áudio correta (monitor do sistema - áudio de saída)
AUDIO_DEVICE=$(pactl list short sources | grep "monitor" | grep -v "hdmi\|bluez" | head -1 | awk '{print $2}')

# Se não encontrar, tenta o padrão
if [ -z "$AUDIO_DEVICE" ]; then
    AUDIO_DEVICE=$(pactl list short sources | grep "monitor" | head -1 | awk '{print $2}')
fi

echo "🎤 Áudio: $AUDIO_DEVICE"

# Grava 15 segundos com áudio
ffmpeg -f x11grab -r 30 -s ${WIDTH}x${HEIGHT} -i :0.0${OFFSET} \
       -f pulse -i "$AUDIO_DEVICE" \
       -t 15 \
       -c:v libx264 -preset fast -crf 20 \
       -c:a aac -b:a 192k \
       -async 1 \
       "$OUTPUT_DIR/Conquista_${TIMESTAMP}.mp4"

# Verifica se gravou com sucesso
if [ $? -eq 0 ]; then
    echo "✅ Gravação salva: $OUTPUT_DIR/Conquista_${TIMESTAMP}.mp4"
    notify-send "🏆 Conquista gravada!" "Vídeo salvo com sucesso" --icon=video
else
    echo "❌ Erro na gravação"
    notify-send "❌ Erro" "Falha ao gravar conquista" --icon=error
fi
```
