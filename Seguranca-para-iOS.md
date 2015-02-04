
Segurança para iOS
==

![System Security Image](https://lh6.googleusercontent.com/43LIlvnoAaekpb4RvL_ximDiNrCLSJAa0klXeCtWgws=s400 "security")

Semana passada eu dei uma palestra sobre segurança em aplicações iOS. O arquivo da apresentação está disponível no [meu blog](http://blog.garrefa.com.br/palestra-seguranca-em-aplicacoes-ios/).

Após a palestra, falei com algumas pessoas e percebi que seria interessante escrever alguns artigos explicando melhor os pontos que foram abordados na palestra e este é o primeiro desses artigos.

Para este artigo escrevi uma aplicação de exemplo simples. A aplicação está disponível via github: https://github.com/garrefa/evil-app

> git clone https://github.com/garrefa/evil-app

Logs (NSLog)
--

Logs são muito úteis no desenvolvimento de um app. Podemos adicionar logs em vários pontos do código e assim ter uma visão clara do que está acontecendo no app, quais dados estão sendo passados, quais funções estão sendo chamadas. 
O problema dos logs é que muitas vezes o app vai pra produção e os logs ficam. Acredite ou não, existem casos famosos de apps fazendo log de usuário e senha na hora do login...

**Mas o processo de deploy pra Apple Store não remove os logs do meu código?** 

*Não...*

**Ok, mas como os logs são capturados? Como alguém que não tem o código do app pode ver os logs em produção? **

É simples. Basta plugar o device em um mac, abrir o Xcode, ir no menu `Window`, `Devices`. Ver imagem abaixo.

![Xcode devices menu](https://lh3.googleusercontent.com/huaXsAgP0JekCZLujU2tlYvfXyxX4IeEFndoXSd-XEQ=s400 "04_devices_xcode_menu.png")

Ao clicar em `Devices` será aberta a tela com a listagem de devices na esquerda. Selecione o device desejado e na direita serão apresentadas as informações deste, aplicativos instalados e logs.

Na imagem abaixo é possível ver a tela de devices aberta e o log específico da aplicação de testes. É possível ver no log o username, senha e número de cartão de crédito do usuário. 

![Xcode logs](https://lh3.googleusercontent.com/niOW21WETE4PtLoQrqpxzuO_eNXb4ucegnrjhtoeNOk=s400 "xcode_logs.png")

A aplicação de testes está versionada. A versão 1.0 pode ser usada para testes em produção e desenvolvimento. Para isso basta fazer checkout na tag v1.0.

> git checkout v1.0

Podemos facilmente evitar que o log vaze para produção usando uma macro no arquivo `pch`. Essa macro vai redefinir NSLog nos casos onde a variável DEBUG não está setada. O código pra isso é muito simples:

>  #ifndef DEBUG
>  #define NSLog(...)
>  #endif

Se estivermos em ambiente de DEBUG, essa três linhas redefinem NSLog para não fazer nada, já que o segundo parâmetro da macro que define NSLog está vazio.

Fazendo checkout na versão 2.0 do app de testes podemos ver esse código aplicado no arquivo `PrefixHeader.pch`.

> git checkout v2.0

No mundo perfeito, o programador adicionaria logs build de desenvolvimento e removeria no build de produção. Alguns programadores dizem que os logs devem ser removidos completamente após uma funcionalidade ser testada. O problema é que remover log é chato, consome tempo e ninguém lembra de fazer. Definir macros ajuda bastante a automatizar esse processo.

Outra possibilidade é usar um pod chamado [CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack). Esse pod possuí vários recursos avançados para controle de logs, logs com cores diferentes para alertas, erros e etc. No futuro deve pintar um artigo por aqui explicando como utilizá-lo.

App Background Snapshots
--

Outra coisa que pouca gente lembra é que quando os apps vão para background o iOS tira um snapshot da tela do app para ser exibida como miniatura. Esse snapshot fica salvo descriptografado no disco e é muito fácil de recuperá-lo. 

Ainda na tela de `Devices` do Xcode, clicar no app desejado, depois no botão com ícone de engrenagem e escolher a opção 'Download Container'. Ver imagem abaixo.

![exporting app container](https://lh5.googleusercontent.com/cchoLEUtjVEMMgi5z4GeO653gf-QhNeA3l8aMUc4p5o=s400 "01_save_container.png")

Salve o container no seu disco e depois troque a extensão para .zip. Fazendo isso é possível navegar na estrutura de diretórios do container e chegar até o snapshot conforme imagem abaixo.

![app snapshot path](https://lh6.googleusercontent.com/GyrCgtFpwOmyWYQLTGuzyNE5fKC3AwYFnyEK-OYv3y8=s400 "03 snapshot path.png")

No caso do app de teste, o hacker não teria visto a senha do usuário pois o campo é protegido, mas ele teria visto e username e o número do cartão de crédito. E como já vimos antes, a senha estava aberta nos logs... 

![app snapshot](https://lh5.googleusercontent.com/r7LobpTgCf_W_tEKs0Rkps0E2XRcMztysTL6mQYBtic=s400 "04 snapshot.png")

**E como eu me protejo desse tipo de ataque?**

Uma forma é colocar uma view por cima da view atual quando o app for entrar em background e depois remover quando ele for voltar ao foreground.
Um exemplo de como fazer isso está na v3.0 do app de testes.

> git checkout v3.0

****

    - (void)applicationDidEnterBackground:(UIApplication *)application {
    	UIView *currentView = [self.window.subviews objectAtIndex:0];
    	UIView *redView = [[UIView alloc]
    	initWithFrame:currentView.bounds];
    	redView.backgroundColor = [UIColor redColor];
    	redView.tag = 100;
    	[currentView addSubview:redView];
    }

****

    - (void)applicationWillEnterForeground:(UIApplication *)application {
        UIView *currentView = [self.window.subviews objectAtIndex:0];
        UIView *redView = [currentView viewWithTag:100];
        [redView removeFromSuperview];
    }

Componentes de Terceiros (pods)
--

Usar pods e componentes de terceiros agiliza muito o trabalho de desenvolvimento. Alguns pods são padrão na área como por exemplo o AFNetworking. É bom lembrar que o mesmo cuidado que temos com o nosso código precisamos ter com código de terceiros, verificar as issues em aberto no git, ver se tem manutenção, se existe uma comunidade de pessoas utilizando. Todo seu esforço de segurança pode ser posto abaixo com um pod em que o desenvolvedor não tomou tanto cuidado com segurança quanto deveria.

**No próximo artigo detalharei o ataque de man in the middle, ataque a redes wifi, funcionamento do SSL, SSL pinning e um pouco de criptografia.**

Uma forma de se proteger é colocar uma outra view por cima da view principal quando o app vai entrar em background.

> Escrito por Alexandre Garrefa (@alexmrg) 
> blog.garrefa.com.br