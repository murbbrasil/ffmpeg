# Repositório de Bibliotecas Android Depreciadas

Repositório para armazenar e distribuir bibliotecas Android (.aar) depreciadas via GitHub Packages da organização murbbrasil.

## Bibliotecas Disponíveis

### FFmpeg Kit Full GPL
- **Versão:** 6.0.LTS
- **Arquivo:** `libs/ffmpeg/ffmpeg-kit-full-gpl-6.0.LTS.aar`
- **Maven:** `com.github.murbbrasil:ffmpeg-kit-full-gpl:6.0.LTS`

## Guia Rápido

### Adicionar Nova Biblioteca
1. Coloque o arquivo `.aar` em uma pasta dentro de `libs/`
2. Adicione uma nova entrada em `publications` no arquivo `build.gradle`:
   ```groovy
   publications {
       novaLib(MavenPublication) {
           artifact file('libs/pasta/arquivo.aar')
           groupId = project.group
           artifactId = 'nome-da-biblioteca'
           version = 'versao'
           // ...
       }
   }
   ```

### Publicar no GitHub Packages

1. **Configure Credenciais**
   
   Você pode usar qualquer uma das opções:
   
   **Opção 1:** Crie ou edite `gradle.properties` na raiz:
   ```
   gpr.user=seu_username_github
   gpr.key=seu_token_github
   ```
   
   **Opção 2:** Crie ou edite `local.properties` na raiz:
   ```
   github.username=seu_username_github
   github.token=seu_token_github
   ```
   
   > 💡 **Dica:** Gere um token com permissões `read:packages` e `write:packages`

2. **Atualize o build.gradle**
   
   Se você escolheu usar o `local.properties`, adicione este código no início do seu `build.gradle`:
   ```groovy
   def localProperties = new Properties()
   def localPropertiesFile = rootProject.file('local.properties')
   if (localPropertiesFile.exists()) {
       localPropertiesFile.withReader('UTF-8') { reader ->
           localProperties.load(reader)
       }
   }
   
   // Em seguida, na seção repositories do publishing:
   publishing {
       repositories {
           maven {
               name = "GitHubPackages"
               url = uri("https://maven.pkg.github.com/murbbrasil/ffmpeg")
               credentials {
                   username = localProperties.getProperty('github.username') ?: System.getenv("GITHUB_USERNAME")
                   password = localProperties.getProperty('github.token') ?: System.getenv("GITHUB_TOKEN")
               }
           }
       }
       // ...
   }
   ```

3. **Execute o Comando**
   ```
   gradlew publish
   ```

4. **Verifique a Publicação**
   
   Após a publicação, verifique em:
   - `https://github.com/orgs/murbbrasil/packages`
   - `https://github.com/murbbrasil/ffmpeg/packages`

### Usar em Projetos Android

1. **Adicione o Repositório** - No arquivo `settings.gradle`:
   ```groovy
   dependencyResolutionManagement {
       repositories {
           // ... outros repositórios
           maven {
               name = "GitHubPackages"
               url = uri("https://maven.pkg.github.com/murbbrasil/ffmpeg")
               credentials {
                   // Você pode usar local.properties
                   def localProps = new Properties()
                   def localFile = new File(rootDir, "local.properties")
                   if (localFile.exists()) {
                       localFile.withInputStream { localProps.load(it) }
                   }
                   username = localProps.getProperty('github.username') ?: findProperty("gpr.user") ?: System.getenv("GITHUB_USERNAME")
                   password = localProps.getProperty('github.token') ?: findProperty("gpr.key") ?: System.getenv("GITHUB_TOKEN")
               }
           }
       }
   }
   ```

2. **Configure Credenciais** - Use um dos métodos:
   
   **Opção 1:** Arquivo `gradle.properties`:
   ```
   gpr.user=seu_username_github
   gpr.key=seu_token_github
   ```
   
   **Opção 2:** Arquivo `local.properties` (recomendado pois já está no .gitignore por padrão):
   ```
   github.username=seu_username_github
   github.token=seu_token_github
   ```

3. **Adicione a Dependência** - No `build.gradle` do módulo:
   ```groovy
   dependencies {
       implementation 'com.github.murbbrasil:ffmpeg-kit-full-gpl:6.0.LTS'
   }
   ```

## Uso do FFmpeg Kit

### Exemplo Básico
```java
import com.arthenica.ffmpegkit.FFmpegKit;

// Converter vídeo para MP4
FFmpegKit.execute("-i input.mp4 -c:v mpeg4 output.mp4");

// Extrair áudio
FFmpegKit.execute("-i input.mp4 -vn -c:a copy output.aac");
```

### Processamento Assíncrono com Callbacks
```java
import com.arthenica.ffmpegkit.FFmpegKit;
import com.arthenica.ffmpegkit.ReturnCode;

FFmpegKit.executeAsync("-i input.mp4 -c:v mpeg4 output.mp4", session -> {
    if (ReturnCode.isSuccess(session.getReturnCode())) {
        // Sucesso
        Log.d("FFmpeg", "Operação completada");
    } else {
        // Falha
        Log.e("FFmpeg", "Erro: " + session.getFailStackTrace());
    }
});
```

## Licença

As bibliotecas seguem suas licenças originais.  
FFmpeg Kit usa a licença [GNU GPL v3.0](https://ffmpeg.org/legal.html).