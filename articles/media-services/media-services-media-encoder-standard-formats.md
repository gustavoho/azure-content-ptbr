<properties 
	pageTitle="Codecs e formatos padrão do Codificador de Mídia" 
	description="Este tópico oferece uma visão geral dos codecs e dos formatos do Codificador de Mídia Padrão." 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/29/2016"
	ms.author="juliako;anilmur"/>

#Codecs e formatos padrão do codificador de mídia


Este documento contém uma lista dos formatos de arquivo de importação e importação mais comuns que você pode usar com o Codificador de Mídia padrão.


##Formatos de arquivo/contêiner de entrada

Formatos de arquivo (extensões de arquivo)|Suportado
---|---|---|---
FLV (com codecs H.264 e AAC) (.flv) |Sim 
MXF (.mxf) |Sim 
GXF (.gxf) |Sim 
MPEG2-PS, MPEG2-TS, 3GP (.ts, .ps, .3gp, .3gpp, .mpg) |Sim 
Vídeo do Windows Media (WMV)/ASF (.wmv, .asf) |Sim 
AVI (8 bits/10 bits descompactado) (.avi)|Sim 
MP4 (.mp4, .m4a, .m4v)/ISMV (.isma, .ismv)|Sim 
[Gravação (DVR-MS) de vídeo Digital da Microsoft](https://msdn.microsoft.com/library/windows/desktop/dd692984) (.dvr-ms) |Sim 
Matroska/WebM (.mkv) |Sim 
WAVE/WAV (.wav) |Sim 
QuickTime (.mov) |Sim

>[AZURE.NOTE] Acima há uma lista das extensões de arquivo encontradas mais comumente. O Media Encoder Standard dá suporte a muitos outros (por exemplo: .m2ts, .mpeg2video, .qt). Se você tentar codificar um arquivo e receber uma mensagem de erro que diz que o formato não tem suporte, forneça comentários [aqui](https://feedback.azure.com/forums/169396-media-services/category/144411-encoding-and-processing/).

###Formatos de áudio em contêineres de entrada 

O Padrão do Codificador de Mídia oferece suporte à execução dos seguintes formatos de áudio em contêineres de entrada:

- Arquivos do MXF, GXF e QuickTime que têm faixas de áudio com exemplos em estéreo intercalado ou de 5.1

ou o

- Arquivos MXF, GXF e QuickTime onde o áudio é executado como faixas PCM separadas, mas o mapeamento de canal (para estéreo ou 5.1) pode ser deduzido dos metadados do arquivo

Observe que o suporte para mapeamento de canal explícito/fornecido pelo usuário será fornecido em um futuro próximo.


##Codecs de vídeo de entrada

Codecs de vídeo de entrada|Suportado
---|---|---|---
AVC de 8 bits/10 bits até 4:2:2, incluindo AVCIntra |8 bits 4:2:0 e 4:2:2 
DNxHD ávido (em MXF) |Sim 
DVCPro/DVCProHD (em MXF) |Sim 
Vídeo digital (VD) (em arquivos AVI) |Sim
JPEG 2000 |Sim 
MPEG-2 (até perfil e de alto nível 422; incluindo variantes como XDCAM, XDCAM HD, XDCAM IMX, CableLabs ® e D10)|Até perfil 422 
MPEG-1 |Sim 
VC-1/WMV9 |Sim 
Canopus HQ/HQX |Não 
MPEG-4, parte 2 |Sim 
[Theora](https://en.wikipedia.org/wiki/Theora) |Sim 
YUV420 descompactado, ou mezzanine |Sim
Apple ProRes 422 |Sim
Apple ProRes 422 LT |Sim
Apple ProRes 422 HQ |Sim
Apple ProRes Proxy|Sim
Apple ProRes 4444 |Sim
Apple ProRes 4444 XQ |Sim



##Codecs de áudio de entrada

Codecs de áudio de entrada|Suportado
---|---|---|---
AAC (AAC-LC, AAC-HE e AAC-HEv2; até 5.1)|Sim 
MPEG Layer 2|Sim 
MP3 (MPEG-1 Audio Layer 3)|Sim 
Áudio do Windows Media|Sim 
WAV/PCM|Sim 
[FLAC](https://en.wikipedia.org/wiki/FLAC)</a>|Sim 
[Opus](https://en.wikipedia.org/wiki/Opus_(audio_format) |Sim 
[Vorbis](https://en.wikipedia.org/wiki/Vorbis)</a>|Sim 
AMR (multitaxa adaptável)|Sim
AES (SMPTE 331M e 302M, AES3-2003) |Não 
Dolby® E |Não 
Dolby® Digital (AC3) |Não 
Dolby® Digital Plus (E-AC3) |Não 


##Formatos e codecs de saída

A tabela a seguir lista os codecs e formatos de arquivo com suporte para exportação.


Formato de arquivo|Codec de vídeo|Codec de áudio
---|---|---
MP4 <br/><br/>(incluindo contêineres MP4 de múltiplas taxas de bits) |H.264 (Perfis Alto, Principal e Linha de base)|AAC-LC, HE-AAC v1, HE-AAC v2 
MPEG2-TS |H.264 (Perfis Alto, Principal e Linha de base)|AAC-LC, HE-AAC v1, HE-AAC v2 



##Roteiros de aprendizagem dos Serviços de Mídia

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##Fornecer comentários

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

##Consulte também

[Codificando conteúdo sob demanda com os Serviços de Mídia do Azure](media-services-encode-asset.md)

[Como codificar com o Codificador de Mídia Padrão](media-services-dotnet-encode-with-media-encoder-standard.md)

<!---HONumber=AcomDC_0706_2016-->