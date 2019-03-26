## ELMAH ##

[ELMAH](http://elmah.github.io/ "Elmah") (Error Logging Modules And Handlers) é uma ferramenta cujo o propósito é logar e reportar exceções não tratadas em aplicações WEB ASP.Net.  
Ela é gratuita, open-source e, infelizmente, pouco conhecida pela maioria.

A partir do momento que o ELMAH é instalado no seu projeto, você recebe as seguintes vantagens sem precisar alterar uma linha de código:

- Logar todas as exceções não tratadas.
- Visualizar todos os erros em detalhes através de uma interface web.
- Receber por e-mail cada exceção disparada em seu site no momento exato em que ocorre. Inclusive uma cópia da "tela amarela da morte" da exceção.

É uma ferramenta extremamente útil e necessária em qualquer aplicação ASP.NET permitindo visualizar e corrigir erros o mais cedo possível.

### Instalação ###

Para aplicações MVC, o mais prático é instalar o pacote Elmah.MVC

    Install-Package Elmah.MVC

Ele já prepara todo o esquema de rotas e configurações no Web.config.

### Testando ###

Vamos criar um projeto demo e forçar uma exceção:

![alt text](http://git.SITE.com.br:8000/tutoriais/elmah/raw/master/Resources/Screenshot_1.png "Imagem 1")

Aqui vemos que o usuário recebera a [Yellow Screen Of Death](https://en.wikipedia.org/wiki/Screen_of_death#Other_screens_of_death "Yellow Screen Of Death")

![alt text](http://git.SITE.com.br:8000/tutoriais/elmah/raw/master/Resources/Screenshot_2.png "Imagem 2")

Ao acessar o endereço http://(server)/elmah, conseguimos visualizar, de forma amigável, todas as exceções não tratadas lançadas pela aplicação até o momento.

![alt text](http://git.SITE.com.br:8000/tutoriais/elmah/raw/master/Resources/Screenshot_3.png "Imagem 3")

Perceba que a forma de armazenamento dos erros é em memória. Ou seja, volátil:

![alt text](http://git.SITE.com.br:8000/tutoriais/elmah/raw/master/Resources/Screenshot_4.png "Imagem 4")

É possível configurar para salvar os logs em arquivos XML ou diretamente em um banco de dados.

Por agora, vamos configurar para salvar em arquivos XML.

Abra o arquivo Web.config do projeto e procure a tag:

    <elmah></elmah>

Modifique para:

    <elmah>
    	<errorLog type="Elmah.XmlFileErrorLog, Elmah" logPath="~/App_Data" />
    </elmah>

Pronto, as novas exceções serão salvas no *logPath* indicado.  
Um arquivo XML por erro.

![alt text](http://git.SITE.com.br:8000/tutoriais/elmah/raw/master/Resources/Screenshot_5.png "Imagem 5")

![alt text](http://git.SITE.com.br:8000/tutoriais/elmah/raw/master/Resources/Screenshot_6.png "Imagem 6")

Para que os erros tambem sejam enviados por email, adicione a tag *errorMail*

    <elmah>		
    	<errorMail from="Error SITE &lt;no-reply@agencia1.com.br>" to="edumonteiro.si@gmail.com" />
    	<errorLog type="Elmah.XmlFileErrorLog, Elmah" logPath="~/App_Data" />
    </elmah>

Ela irá utilizar as configurações de SMTP setadas no Web.config.

> Se você não sabe configurar o smtp no web.config, seria algo como:
> 
>     <system.net>
> 		<mailSettings>
> 			<smtp from="Error SITE &lt;no-reply@agencia1.com.br>">
> 				<network host="smtp.agencia1.com.br" port="587" userName="no-reply@agencia1.com.br" password="123mudar!" enableSsl="false" />
> 			</smtp>
> 		</mailSettings>
> 	</system.net>



Com isso, você, alem de loggar os erros em arquivo XML, também receberá uma cópia por e-mail no momento exato que o mesmo acontecer.

E por fim, para acessar a url do elmah (http://server/elmah) externamente, é necessario adicionar a tag security a seção do elmah:

    <elmah>		
    	<errorMail from="Error SITE &lt;no-reply@agencia1.com.br>" to="edumonteiro.si@gmail.com" />
    	<errorLog type="Elmah.XmlFileErrorLog, Elmah" logPath="~/App_Data" />

		<security allowRemoteAccess="1" />
    </elmah>

Atenção, dessa forma, qualquer um poderá acessar a URL. De forma alguma isso é seguro. Pode haver informações sensíveis expostas em alguns erros.

Vamos proteger a URL para acesso somente a pessoas autenticadas/autorizadas.

Ainda no Web.config, em *appSettings*, altere a chave requiresAuthentication para o valor *true* e indique as roles e/ou os usuários que podem visualizar a URL:


    <add key="elmah.mvc.requiresAuthentication" value="true" />
  	<add key="elmah.mvc.allowedRoles" value="Admin" />
    <add key="elmah.mvc.allowedUsers" value="*" />


### Filtrando erros ###

Dependendo do projeto pode ser bem chato e custoso receber por e-mail qualquer erro404 que venha a acontecer na aplicação (podem ser muitos).

Podemos filtrar pra não enviar esse tipo de erro por e-mail. Basta adicionar um filtro:

    <elmah>		
		<errorFilter>
      		<test>
          		<and>
		            <equal binding="HttpStatusCode" value="404" type="Int32" />
	        	    <regex binding="FilterSourceType.Name" pattern="mail" />
	          	</and>
      		</test>
		</errorFilter>

    	<errorMail from="Error SITE &lt;no-reply@agencia1.com.br>" to="edumonteiro.si@gmail.com" />
    	<errorLog type="Elmah.XmlFileErrorLog, Elmah" logPath="~/App_Data" />

		<security allowRemoteAccess="1" />
    </elmah>


É isso.