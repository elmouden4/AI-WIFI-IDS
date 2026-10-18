# Mini IDS Wi-Fi avec IA sur PC et terminal ESP32

Projet de Master — prototype pédagogique, documentation en français.

**Commencer par [DEMARRAGE.md](docs/DEMARRAGE.md).** Le code cible un ESP32 DevKit classique
et un OLED **SSD1306 I2C 128 × 64, 0,96 pouce, quatre broches**. La taille seule ne garantit
pas le contrôleur : vérifier la référence du module. Un SH1106 ou un écran SPI demande une adaptation.

## Livrables

- `python/train.py` : Random Forest, nettoyage, séparation train/test, métriques JSON et modèle joblib.
- `python/predict.py` : classification des CSV CICFlowMeter, journal CSV, USB série facultatif.
- `python/capture.py` : capture passive TShark en PCAP et contrôle de présence de paquets IP.
- `python/send_esp32.py` : démonstration des états, liste des ports et transport acquitté.
- `esp32/esp32_alert/esp32_alert.ino` : OLED, LEDs, buzzer, délai de perte de données.
- [Montage](docs/MONTAGE.md), [capture et compatibilité](docs/CAPTURE.md),
  [trame du mémoire](docs/RAPPORT_MASTER.md), [validation](docs/VALIDATION.md).
- `tests/test_pipeline.py` : tests automatisés indépendants du matériel.

## Architecture

```text
Trafic IP du laboratoire
        │
Alfa / interface réseau → TShark → fichier PCAP
                                    │
                         CICFlowMeter (outil externe)
                                    │
                             CSV de flux finalisé
                                    │
                   Python + Random Forest sur le PC
                         │                    │
                   journal CSV         USB série 115200
                                              │
                                            ESP32
                                    OLED + LEDs + buzzer
```

La conversion PCAP → CSV se fait dans CICFlowMeter, installé séparément. Cette version
fonctionne par captures terminées et relecture de flux ; elle ne prétend pas fournir une
surveillance continue en temps réel. Le rythme de relecture du CSV n'est pas celui du réseau.
Le PC exécute l'IA ; l'ESP32 est un terminal embarqué. Présenter le projet comme une
**architecture hybride de détection et supervision embarquée**. Pour une exigence académique
d'inférence embarquée, prévoir une seconde phase TinyML ou une cible Linux embarquée.

## États

| État | Sens | Sorties |
|---|---|---|
| NORMAL | Flux prédit bénin | LED verte |
| DOS | Flux suspect DoS/DDoS | LED rouge et buzzer intermittent |
| SCAN | Flux suspect PortScan | LED rouge et buzzer intermittent |
| UNKNOWN | Score faible, données invalides, attente ou fin | LEDs éteintes, buzzer éteint |

Une absence de messages pendant 5 secondes affiche `LINK / DATA TIMEOUT`.
L'alerte est conservée au moins 3 secondes avant l'acceptation d'un nouveau NORMAL ;
UNKNOWN la réinitialise immédiatement. Les prédictions portent sur les flux observés,
pas sur la sécurité globale du réseau. Le score Random Forest n'est pas une probabilité calibrée.

## État de livraison

Code et documentation fournis ; tests Python exécutés sur données artificielles.
Le dataset CIC-IDS2017 et un modèle entraîné sur données réelles ne sont pas inclus.
Le firmware n'a pas été compilé ici (chaîne Arduino ESP32 absente), ni testé sur une carte.
La capture Alfa et l'export CICFlowMeter nécessitent une validation sur votre installation.
Aucun résultat de précision sur CIC-IDS2017 n'est inventé.

## Sources techniques

- [CIC-IDS2017, UNB](https://www.unb.ca/cic/datasets/ids-2017.html) : données de flux étiquetées.
- [CICFlowMeter officiel](https://github.com/ahlashkari/CICFlowMeter) : conversion des captures.
- [TShark](https://www.wireshark.org/docs/man-pages/tshark.html) : options de capture.
- [I2C ESP32](https://docs.espressif.com/projects/arduino-esp32/en/latest/api/i2c.html) : bus I2C.
- [Adafruit SSD1306](https://github.com/adafruit/Adafruit_SSD1306) : bibliothèque OLED.
- [RandomForestClassifier](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html).
"# AI-WIFI-IDS" 
