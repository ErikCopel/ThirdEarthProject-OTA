# Gravação de firmware — ThirdEarthProject

Este guia descreve o flash do gateway e do medidor ESP32-S3 por USB e a diferença entre firmware parcial e firmware full.

## Firmware parcial e firmware full

| Tipo | Conteúdo | Uso principal | Configurações do dispositivo |
| --- | --- | --- | --- |
| **Parcial** | Somente a aplicação | Atualizações via OTA em dispositivos já instalados | Preserva as demais partições; a compatibilidade das configurações depende da versão instalada |
| **Full** | Bootloader, tabela de partições, aplicação e dados iniciais necessários | Gravação em fábrica, primeira instalação e recuperação por USB | No procedimento de fábrica abaixo, as partições são apagadas e é necessário reconfigurar o dispositivo |

O **firmware parcial é utilizado principalmente para updates via OTA**. Ele depende do bootloader e da tabela de partições já existentes e não deve ser gravado no endereço `0x0`. A atualização OTA deve ser executada pela rotina de atualização do dispositivo, quando suportada, usando o binário de aplicação correspondente.

O **firmware full é utilizado principalmente para gravações em fábrica**. O procedimento deste guia apaga toda a memória flash antes de gravar a imagem completa. Isso apaga as partições e seus dados, incluindo configurações persistidas, credenciais e cadastros locais. **Após a gravação full, é necessário reconfigurar o dispositivo.**

A imagem full sozinha sobrescreve as regiões abrangidas por ela; o comando `erase-flash` abaixo garante a limpeza de todas as partições, inclusive as que estejam fora dessas regiões.

## Arquivos de cada dispositivo

| Dispositivo | Firmware parcial — aplicação | Firmware full — gravação em fábrica |
| --- | --- | --- |
| Gateway | `gateway-firmware.bin` | `gateway-firmware-full.bin` |
| Medidor ESP32 | `esp32-meter-firmware.bin` | `esp32-meter-firmware-full.bin` |

Use sempre o arquivo correspondente ao dispositivo. Os comandos abaixo são para placas **ESP32-S3** compatíveis com o projeto. A disponibilidade do binário parcial não significa que a versão instalada no dispositivo tenha suporte a OTA.

## Preparar a gravação por USB

Instale o esptool 5 em um ambiente virtual ou use o ambiente Python do ESP-IDF que já tenha essa versão:

```sh
python3 -m venv .venv
source .venv/bin/activate
python -m pip install 'esptool>=5,<6'
```

No Windows, ative o ambiente com `.venv\Scripts\activate` no Prompt de Comando ou `.venv\Scripts\Activate.ps1` no PowerShell.

Conecte o dispositivo com um cabo USB de dados e feche qualquer monitor serial. No macOS, liste as portas:

```sh
ls /dev/cu.*
```

Substitua `/dev/cu.usbmodemXXXX` nos comandos pela porta real: por exemplo, `/dev/ttyACM0` ou `/dev/ttyUSB0` no Linux e `COM5` no Windows. Execute os comandos na pasta que contém os binários.

## Gravação full do gateway

**Este procedimento apaga todas as partições e configurações do gateway.**

```sh
python -m esptool --chip esp32s3 --port /dev/cu.usbmodemXXXX erase-flash
python -m esptool --chip esp32s3 --port /dev/cu.usbmodemXXXX write-flash 0x0 gateway-firmware-full.bin
```

## Gravação full do medidor ESP32

**Este procedimento apaga todas as partições e configurações do medidor.**

```sh
python -m esptool --chip esp32s3 --port /dev/cu.usbmodemXXXX erase-flash
python -m esptool --chip esp32s3 --port /dev/cu.usbmodemXXXX write-flash 0x0 esp32-meter-firmware-full.bin
```

## Finalizar e reconfigurar

Aguarde a conclusão da gravação e da verificação. Pressione **RESET**, se necessário, e realize novamente a configuração e o cadastro exigidos pelo firmware do dispositivo, incluindo a conexão Wi-Fi no gateway quando aplicável.

Se a ferramenta não conseguir conectar, segure **BOOT**, pressione e solte **RESET** e depois solte **BOOT**. Confira a porta novamente e repita o comando. Se o dispositivo reiniciar após o apagamento, entre novamente no modo de download antes de gravar a imagem full.
