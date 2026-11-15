# SETUP_DEV.md — Ambiente Portátil (Sem Admin) e Servidor Gratuito

## 1. Setup nos PCs dos Estagiários (16 GB RAM / Sem Permissão Admin)

Nenhum instalador `.exe` ou `.msi` deve ser usado. Extraia tudo dentro de uma pasta no diretório do usuário, ex.: `C:\Users\<seu_usuario>\DevMinecraft\`.

1. **Java 17 Portátil:**
   * Baixe o arquivo `.zip` do **Eclipse Temurin JDK 17 (Windows x64)** e extraia em `DevMinecraft\jdk-17\`.
2. **IDE Portátil:**
   * Baixe o arquivo `.zip` do **IntelliJ IDEA Community Edition** e extraia em `DevMinecraft\intellij\`.
   * Em `File -> Project Structure -> SDKs`, aponte para `DevMinecraft\jdk-17`.
3. **Executar o Jogo Sem o Launcher Oficial:**
   * Abra o terminal integrado do IntelliJ na pasta do repositório e execute:
     ```bash
     ./gradlew runClient
     ```
   * O Gradle baixará os binários oficiais da Mojang para o cache local. Quando a janela do jogo abrir (ou via plugin *DevAuth* configurado no `build.gradle`), faça login normalmente com sua **conta Microsoft que possui o Minecraft Java comprado**.
4. **Para Jogar/Testar o Modpack Completo (Fora da IDE):**
   * Baixe o **Prism Launcher (Windows Portable `.zip`)**, extraia em `DevMinecraft\prism\`, crie uma instância **1.20.1 Forge**, importe os mods de `MODPACK_MANIFEST.md` e limite a alocação de RAM da instância para **6144 MB (6 GB)** para deixar folga confortável nos 16 GB do sistema.

---

## 2. Hospedagem Gratuita do Servidor Multiplayer (Oracle Cloud Always Free)

Para evitar sobrecarregar o PC pessoal da equipe e permitir testes 24/7 com vários jogadores:

1. Crie uma conta na **Oracle Cloud Infrastructure (OCI)** (nível *Always Free*).
2. Crie uma instância **VM.Standard.A1.Flex (Ampere ARM)** configurada dentro do limite gratuito:
   * **OCPUs:** `4`
   * **Memória RAM:** `24 GB`
   * **Sistema Operacional:** Ubuntu 22.04 LTS
3. Libere a porta `25565` (TCP/UDP) na *Security List* da VCN da Oracle e no `ufw` do Ubuntu.
4. Instale o Java 17 ARM (`sudo apt install openjdk-17-jdk-headless`), suba o servidor Forge 1.20.1 com **16 GB de RAM dedicados** (`-Xmx16G -Xms8G` com flags Aikar/G1GC) e deixe o servidor rodando em sessão `tmux`.

### Alternativa Rápida (Zero Configuração de Nuvem)
* Para testes pontuais de 2 a 4 pessoas durante uma call, o Tech Lead pode abrir o mundo localmente e usar o mod **e4mc** (ou túnel **Playit.gg** portátil), gerando um endereço público temporário sem precisar abrir portas no roteador.
