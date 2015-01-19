# Git, Xcode e Testes Automatizados
## como criar valida��es b�sicas para comandos do git

O Git vem se tornando cada vez mais importante nas minhas atividades do dia a dia. Tenho utilizado para backup e sincronia das minhas configura��es de ambiente, *.bash\_profile*, *.gitignore\_global*, *.gitconfig*, *.emacs/* e outros. Al�m disso, obviamente utilizo como reposit�rio dos meus projetos e aqui no trabalho, utilizamos o Git diariamente como sistema de controle de vers�o dos nossos projetos.

Utilizando o Git todos os dias, surgem necessidades espec�ficas que nos for�am a fugir do b�sico: clone, commit, pull, push, merge.

Um dos projetos que estou trabalhando atualmente possui tr�s targets. Tenho o target de produ��o, o target ad hoc para nosso sistema interno de distribui��o de testes e o target de testes do calaba.sh, al�m � claro do nosso target de testes unit�rios. Com tantos targets e quatro desenvolvedores trabalhando no mesmo c�digo, � normal que vez por outro algu�m crie uma nova classe ou resource no projeto e esque�a de adicionar em todos os targets necess�rios. Depois de quebrar os testes automatizados do calaba.sh muitas vezes e ter que ca�ar qual a classe/resource que foi esquecida, resolvi escrever um script que checa se alguma das classes/resources adicionadas a algum target do projeto est� presente em todos os outros targets onde ela deveria ter sido adicionada.

### Git Hooks 

Hooks s�o scripts de verifica��o atrelados a algum comando do Git. Nesse artigo estamos falando especificamente de hooks no lado do cliente. � poss�vel configurar hooks para commits, pushs, apply, rebase, e outros. � poss�vel por exemplo, checar se um commit est� seguindo o template padr�o. No nosso caso, a ideia � que sempre que eu fizer um git push, o git pare, rode meu script de valida��o checando se as classes foram adicionadas nos targets corretamente e s� continue se o script finalizar com sucesso ou que eu opte por seguir mesmo assim.

### Download e Instala��o dos Scripts

Supondo que meu projeto esteja no caminho *~/projetos/my-ios-project*, faremos:

	$ cd ~/projetos/my-ios-project
	$ git clone git@github.com:garrefa/ci-scripts.git scripts
	$ cp scripts/pre-push.sample .git/hooks/pre-push

A ativa��o dos hooks � muito simples, basta copiar o script de hook para a pasta *.git/hooks* com o nome correto e est� feito. Esse diret�rio contem v�rios arquivos *.sample* de exemplos de hooks para coisas diferentes, recomendo listar e dar uma olhada nesses arquivos.

### Funcionamento

Se analisarmos o script de hook de pre-push veremos que ele executa um outro script:

	$ cat .git/hooks/pre-push
	...
	...
	python3 "../../scripts/validate-pbx.py"

	if [ $? -gt 0 ]
	then
		read -p "Faltam resources em alguns targets. Confirmar o push? [s|n] " -n 1 -r < /dev/tty
		echo
		if echo $REPLY | grep -E '^[Ss]$' > /dev/null
		then
			exit 0 # push will execute
		fi
		exit 1 # push will not execute
	else
		exit 0 # push will execute
	fi

Com isso podemos ver que o script de pre-push vai chamar o *valdate-pbx.py*. Este script ser� encerrado com um status code que indica se a valida��o teve sucesso ou n�o. Status code = 0 indica que o script finalizou com sucesso e qualquer coisa maior que zero indica que houve erro.

O script de pre-push analisa o status code, caso seja maior que zero (-gt = greater than), ent�o exibe uma mensagem para o usu�rio indicando que houveram erros na valida��o e solicita confirma��o do push. Se o usu�rio digitar S ou s, entao o script encerra retornando 0. Isso indica ao hook que o push pode continuar normalmente. Caso o usu�rio digite qualquer coisa diferente de S ou s, ent�o o script finaliza com 1 e o push � cancelado.
� poss�vel for�ar a quebra do push em situa��es de erro e n�o oferecer ao usu�rio a op��o de continuar mesmo assim. Para isso basta substituir esse script todo por algo assim:


	python3 "../../scripts/validate-pbx.py"

	if [ $? -gt 0 ]
	then
		exit 1 # push will not execute
	else
		exit 0 # push will execute
	fi

### Bonus

Ok. Agora voc� configurou o hook, o script valida antes de permitir o push, mas... Voc� esqueceu de adicionar uma classe a algum target, vai ter que ca�ar qual a classe? Felizmente n�o. Em caso de erros, o script *validate-pbx.py* imprime a lista dos targets relevantes e cada classe que est� faltando. Vamos a um exemplo:

	$ ./scripts/validate-pbx.py

	[4] files in my-ios-project-adhoc.app
	[Missing] : * Modernizador.m in Sources
	[Missing] : * UIImage+AX.m in Sources
	[Missing] : * BarCodeReaderViewController.m in Sources
	[Missing] : * DateSelectorView.m in Sources

### Script Source

Os scripts descritos neste artigo est�o no meu github: git@github.com:garrefa/ci-scripts.git
Se voc� usou/l�u e gostou, mande um ping no twitter @alexmrg

Alexandre Garrefa : @alexmrg

alexandre.garrefa@concretesolutions.com.br

iOS Developer - Concrete Solutions
