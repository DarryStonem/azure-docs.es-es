---
title: Bing Speech API in Microsoft Cognitive Services | Microsoft Docs
description: Use the Bing Speech API to add speech-driven actions to your apps, including real-time interaction with users.
services: cognitive-services
author: priyaravi20
manager: yanbo
ms.service: cognitive-services
ms.technology: speech
ms.topic: article
ms.date: 02/28/2017
ms.author: prrajan
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: 4831c26379d27a9a47eec01d2d3d76a67f4e51b4
ms.contentlocale: es-es
ms.lasthandoff: 05/10/2017

---
# <a name="bing-speech-api-overview"></a>Bing Speech API Overview
Welcome to Speech Service – Microsoft’s speech recognition and text to speech cloud offering which helps users to voice enable their applications and bring in a delightful conversational experience. 

Microsoft Speech service has two APIs:
* Speech To Text  :   
    Using REST API : For apps converting short spoken commands(upto 12 seconds) to text without real-time streaming or user feedback.
  Using Websocket API : For apps converting long audio(upto 10 minutes) to text with intermediate results.
* Text To Speech API : For apps converting text into audio that can be played back to the user. 

## <a name="speech-to-text-speech-recognition"></a>Speech to text (Speech Recognition)
Microsoft's Speech Service *transcribes* audio streams into text suitable for display to a user. Transcription includes adding appropriate capitalization and punctuation, masking profanity, and normalizing text. For example, if a user says `remind me to buy six pencils`, Microsoft's Speech Services will return the transcribed text `Remind me to buy 6 pencils.` There are two options for adding speech capabilities to your app:
* The REST API uses chunked-transfer encoding to convert short spoken commands without real-time streaming or user feedback.
* The WebSocket API uses full-duplex communication to convert longer audio input and supports intermediate results.
Use this comparison chart to help you choose the API that fits your needs.
### <a name="quick-feature-comparison"></a>Quick feature comparison
| Feature | WebSocket API | REST API |
|-----|-----|-----|
|Continuous Recognition | Yes | No |
| Audio Length  | 10 mins | 15 seconds |
| Partial results | Yes | No |
| Service Endpointing | Yes| No |
| Recognition Modes | Interactive, Dictation, Conversational| Interactive, Dictation, Conversational |
| Debug support with Telemetry | Yes | No |
| Subscription Key Authorization | Yes | No |
| Reco Results | Lexical, Itn, Masked | Lexical, Itn, Masked |
| N-Best | Up to 5 | Up to 5 |
| Confidence | Yes| Yes |  

## <a name="supported-languages"></a>Supported languages  
Locales supported by the speech to text (recognition) API include:    

|language-Country |language-Country | language-Country |language-Country   
|---------|----------|--------|---------------  
|ar-EG    |  en-NZ  | it-IT  |  ru-RU  
|ca-ES    |  en-US  | ja-JP  |  sv-SE  
|da-DK    |  es-ES  | ko-KR  |  zh-CN  
|de-DE    |  es-MX  | nb-NO  |  zh-HK  
|en-AU    |  fi-FI  | nl-NL  |  zh-TW  
|en-CA    |  fr-CA  | pl-PL  |      
|en-GB    |  fr-FR  | pt-BR  |        
|en-IN    |  hi-IN  | pt-PT  |  


## <a name="text-to-speech-speech-synthesis"></a>Text to speech (Speech Synthesis)
The speech synthesis REST API provides real-time text to speech conversion in a variety of different voices and languages. The maximum amount of audio returned for any single request must not exceed 15 seconds. 

### <a name="supported-languages"></a>Supported languages  
Locales supported by the text to speech (synthesis) API include:  

language-Country |language-Country | language-Country | language-Country  
---------|----------|------------|------------  
ar-EG    |   en-GB  |   hi-IN  |   pt-PT  
ar-SA    |   en-IE  |   hu-HU  |   ro-RO  
ca-ES    |   en-IN  |   id-ID  |   ru-RU  
cs-CZ    |   en-US  |   it-IT  |   sk-SK  
da-DK    |   es-ES  |   ja-JP  |   sv-SE  
de-AT    |   es-MX  |   ko-KR  |   th-TH  
de-CH    |   fi-FI  |   nb-NO  |   tr-TR  
de-DE    |   fr-CA  |   nl-NL  |   zh-CN  
el-GR    |   fr-CH  |   pl-PL  |   zh-HK  
en-AU    |   fr-FR  |   pt-BR  |   zh-TW    
 




