# Guia Rápido

Este guia contém instruções passo a passo para criar um aplicativo iOS com o Watson Developer Cloud iOS SDK. O aplicativo que construimos irá sintetizar um texto em Inglês em um audio falado utilizando o serviço Watson Text to Speech.

## Criar o Aplicativo

1. Abrir o Xcode e criar um novo projeto com o modelo "Single View Application".

    ![New Project](quickstart-resources/01-NewProject.png?raw=true)

2. Defina o nome do produto para "Watson Speaks" e a linguagem para "Swift".

    ![Project Settings](quickstart-resources/02-ProjectSettings.png?raw=true)

## Download e Construção do Framework iOS SDK

Nós iremos utilizar o gerenciador de dependências [Carthage](https://github.com/Carthage/Carthage) para realizar o download e a construção do iOS SDK. Você precisará [instalar o Carthage](https://github.com/Carthage/Carthage#installing-carthage) se ainda não estiver instalado em seu sistema.

1. Crie um arquivo no diretório do projeto chamado de `Cartfile`.

    ![Create Cartfile](quickstart-resources/03-CreateCartfile.png?raw=true)

2. Adicione a seguinte linha ao `Cartfile`. Isso especifica o SDK do iOS como uma dependência. Em um aplicativo de produção, você também poderá especificar um [requisito de versão](https://github.com/Carthage/Carthage/blob/master/Documentation/Artifacts.md#version-requirement).

    ```
    github "watson-developer-cloud/ios-sdk"
    ```

    ![Add Dependency](quickstart-resources/04-AddDependency.png?raw=true)

3. Abra o Terminal e navegue até o diretório do projeto. Em seguida, use Carthage para fazer o download e construir o iOS SDK:

    ```
    $ carthage update --platform iOS
    ```

    ![Carthage Update](quickstart-resources/05-CarthageUpdate.png?raw=true)

Carthage clona o repositório SDK do iOS e constrói o frameword para cada serviço Watson no diretório `Carthage / Build / iOS`. Ele também constrói uma estrutura chamada `RestKit` que é usada internamente para rede/comunicação e parsing JSON.

### Adicionar o Framework iOS SDK ao Aplicativo

Para usar o frameword do iOS SDK, precisamos vinculá-las ao nosso aplicativo.

1. No Xcode, navegue até a guia de configurações "Geral" do seu aplicativo. Em seguida, role até a seção "Linked Frameworks and Libraries" e clique no ícone `+`.

    ![General Settings](quickstart-resources/06-GeneralSettings.png?raw=true)

2. Na janela que aparece, escolha "Adicionar outro ..." e navegue até o diretório `Carthage / Build / iOS`. Selecione os frameworks `TextToSpeechV1` e` RestKit` para então vinculá-las à sua aplicação.

    ![Link Frameworks](quickstart-resources/07-LinkFrameworks.png?raw=true)

Também precisamos copiar os frameworks em nosso aplicativo para torná-los acessíveis em tempo de execução. Usaremos um script do Carthage para copiar os frameworks e evitar o [bug de submissão da App Store](http://www.openradar.me/radar?id=6409498411401216).

1. No Xcode, navegue até a guia de configurações do "Build Phases" do seu aplicativo. Em seguida, clique no ícone `+` e adicione uma "New Run Script Phase.".

    ![New Run Script Phase](quickstart-resources/09-NewRunScriptPhase.png?raw=true)

2. Adicione o seguinte comando para executar a fase de script:

    ```
    /usr/local/bin/carthage copy-frameworks
    ```

    ![AddCarthageScript](quickstart-resources/10-AddCarthageScript.png?raw=true)

4. Adicione os frameworks que você gostaria de usar (e o framework `RestKit`) para a lista" Input Files ":

    ```
    $(SRCROOT)/Carthage/Build/iOS/TextToSpeechV1.framework
    $(SRCROOT)/Carthage/Build/iOS/RestKit.framework
    ```

    ![Add Input Files](quickstart-resources/11-AddInputFiles.png?raw=true)

## Adicionar exceção para a Segurança de Transporte de Aplicativo

Para se conectar de forma segura aos serviços do IBM Watson, o arquivo `Info.plist` do aplicativo deve ser modificado com uma exceção de Segurança de Transporte de Aplicativo para o domínio `watsonplatform.net`.

1. No Xcode, clique com o botão direito no arquivo `Info.plist` e escolha `Open As -> Source Code`.

    ![Open As Source Code](quickstart-resources/12-OpenAsSourceCode.png?raw=true)

2. Copie e cole o seguinte código-fonte no arquivo `Info.plist`.

    ```
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSExceptionDomains</key>
        <dict>
            <key>watsonplatform.net</key>
            <dict>
                <key>NSTemporaryExceptionRequiresForwardSecrecy</key>
                <false/>
                <key>NSIncludesSubdomains</key>
                <true/>
                <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
                <true/>
                <key>NSTemporaryExceptionMinimumTLSVersion</key>
                <string>TLSv1.0</string>
            </dict>
        </dict>
    </dict>
    ```

    ![App Transport Security Exception](quickstart-resources/13-AppTransportSecurity.png?raw=true)

## Sintetizar com o Text to Speech

Modificaremos o `ViewController` do nosso projeto para sintetizar p texto em inglês utilizando serviço Text to Speech.

1. No Xcode, abra o arquivo `ViewController.swift`.

2. Adicionar as instruções de importação para `TextToSpeechV1` e` AVFoundation`:

    ```swift
    import TextToSpeechV1
    import AVFoundation
    ```

    ![Import Frameworks](quickstart-resources/14-ImportFrameworks.png?raw=true)

3. Adicione uma propriedade `var audioPlayer: AVAudioPlayer!` à classe `ViewController`. Isso garante que o player de áudio não saia fora do escopo e termina a reprodução quando a função `viewDidLoad ()` retornar.

4. Adicione o seguinte código ao seu método `viewDidLoad`. Certifique-se de atualizar o nome de usuário e a senha com as credenciais para a sua instância do Watson Text to Speech.

    ```swift
    let username = "your-text-to-speech-username"
    let password = "your-text-to-speech-password"
    let textToSpeech = TextToSpeech(username: username, password: password)

    let text = "All the problems of the world could be settled easily if men were only willing to think."
    let failure = { (error: Error) in print(error) }
    textToSpeech.synthesize(text, failure: failure) { data in
        self.audioPlayer = try! AVAudioPlayer(data: data)
        self.audioPlayer.play()
    }
    ```

    ![Synthesize Text](quickstart-resources/15-SynthesizeText.png?raw=true)

5. Execute seu aplicativo no simulador para ouvir o texto sintetizado!
