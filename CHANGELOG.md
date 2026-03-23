# Changelog

Todas as alterações notáveis deste repositório (instalador web + firmware publicado) serão documentadas aqui.

## [1.0.2] — 2026-03-23

### Firmware

- Binário publicado: `firmware/navblue_waveshare_v1.0.2.bin` (substitui `navblue_waveshare_v1.0.1.bin`).
- `manifest.json` atualizado para versão **1.0.2** e caminho do `.bin` correspondente.

### Instalador web (`index.html`)

- Cenário **nenhuma porta selecionada** (`NotFoundError` ao cancelar o seletor serial): ecrã dedicado alinhado à [esp-web-tools](https://github.com/esphome/esp-web-tools) — passos de troubleshooting e links para drivers (Silabs CP2102, WCH CH34x/CH9102), com dica **Linux** (`dialout` / `usermod`) quando aplicável.
- Botões **Cancel** e **Try Again**: mesma largura (grid `1fr 1fr`), mesma altura e modelo de caixa (padding, `appearance: none`, sem conflito com estilos globais).
- `NotAllowedError` mantém-se sem ecrã intrusivo (comportamento anterior).

### Documentação (`README.md`)

- Testes locais: servir a pasta via HTTP (ex.: `python3 -m http.server 8765` → `http://localhost:8765/`); aviso explícito para **não** abrir páginas com `file://` (quebra `fetch` do manifest e do fluxo de instalação).
- App móvel **Navblue** em conjunto com o device HUD: descrição resumida e links para [Google Play](https://play.google.com/store/apps/details?id=com.energuide.navblue) e [App Store](https://apps.apple.com/app/navblue/id6651858865).

---

## [1.0.1] — anterior

- Primeira publicação do binário `firmware/navblue_waveshare_v1.0.1.bin` no instalador web e `manifest.json` correspondente.
