# PHP Senior G5

Esta biblioteca mapeia os serviços da tecnologia [G5 da Senior](https://documentacao.senior.com.br/) em um conjunto de classes PHP.

## Instalação

Adicione **senior-g5-php** no seu projeto via [Composer](https://getcomposer.org/doc/00-intro.md):

```bash
composer require paulosscruz/senior-g5-php
```

## Suporte

Atualmente, esta biblioteca suporta todos os serviços da **tecnologia G5** disponíveis na [documentação oficial](https://documentacao.senior.com.br/) da Senior.

|   Versão     |                Sistema                                                                                            |   Suporte   |
| :---         | :---                                                                                                              | :---        |
|  **5.8.10**  | [ERP Senior](https://documentacao.senior.com.br/gestaoempresarialerp/5.8.10/#webservices/indice-web-services.htm) |   **SIM**   |
|  **5.8.11**  | [ERP Senior](https://documentacao.senior.com.br/gestaoempresarialerp/5.8.11/#webservices/indice-web-services.htm) |   **SIM**   |
|  **5.10.1**  | [ERP Senior](https://documentacao.senior.com.br/gestaoempresarialerp/5.10.1/#webservices/indice-web-services.htm) |   **SIM**   |
|  **6.2.33**  | [VetoRH](https://documentacao.senior.com.br/gestao-de-pessoas-hcm/6.2.33/#webservices/gestao_pessoas.htm)         |   **SIM**   |
|  **6.2.34**  | [VetoRH](https://documentacao.senior.com.br/gestao-de-pessoas-hcm/6.2.34/#webservices/gestao_pessoas.htm)         |   **SIM**   |
|  **6.2.35**  | [VetoRH](https://documentacao.senior.com.br/gestao-de-pessoas-hcm/6.2.35/#webservices/gestao_pessoas.htm)         |   **SIM**   |

## Como funciona

Após o mapear o WSDL do serviço, será gerada uma classe principal que conterá um método para cada porta disponível, juntamente com um conjunto auxiliar de classes que mapeia todos os atributos de entrada e saída de suas respectivas portas.

Para isso, é necessário obter o WSDL do serviço que sera mapeado, você pode encontra-lo na documentação oficial da **Senior**.

Você utilizará o comando `generator` via prompt para mapear o WSDL, abaixo a lista de argumentos disponíveis:

|   Argumento            |                Descrição                                                                                                                                | Obrigatório? |
| :---                   | :---                                                                                                                                                    | :---         |
|  `--wsdl`              | URL WSDL do serviço (Disponível na documentação oficial da Senior).                                                                                     | **SIM**      |
|  `--serviceName`       | Nome do serviço (Este será o nome da classe principal).                                                                                                 | **SIM**      |
|  `--outputDir`         | Diretório onde as classes geradas serão armazenadas (Por padrão, será criado um diretório adional a partir daqui para cada serviço mapeado).            | **SIM**      |
|  `--operationNames`    | Alguns serviços possuem muitas portas diferentes ou versões de uma mesma porta, portanto, aqui você pode especificar quais dessas portas você deseja mapear. | **NÃO**      |
|  `--namespace`         | Especificar o namespace das classes que serão geradas, se não for informado as classes não terão um namespace.                                          | **NÃO**      |

Neste exemplo, vamos utilizar o serviço `com.senior.g5.co.ger.cad.clientes` que possue várias portas, entre elas [`obterCliente`](https://documentacao.senior.com.br/gestaoempresarialerp/5.10.1/#webservices/com_senior_g5_co_ger_cad_clientes.htm#obterCliente) que é capaz de obter o cadastro de um cliente especifico.

### Mapeando o WSDL

```cmd
# php vendor/paulosscruz/senior-g5-php/bin/generator --wsdl http://example.com/g5-senior-services/sapiens_Synccom_senior_g5_co_ger_cad_clientes?wsdl --serviceName Clientes --outputDir senior/services
```

> **ATENÇÃO:** Lembre-se de executar este comando na raiz do seu projeto.


As classes serão geradas no diretório `senior/services/Clientes`, lembrando que a biblioteca sempre cria um diretório adicional a partir de `outputDir`, para separar as classes por **serviço** e evitar problemas de duplicidade.

Neste caso, teremos a classe `Clientes.php` com todos os métodos que representam as portas disponíveis neste serviço. O método `obterClientes` possui a seguinte assinatura:

```php
/**
* @param string $user
* @param string $password
* @param int $encryption
* @param clientesobterClienteIn $parameters
* @return clientesobterClienteOut
*/
public function obterCliente($user, $password, $encryption, clientesobterClienteIn $parameters)
{
    // 
}
```
> As classes `clientesobterClienteIn` e `clientesobterClienteOut` são geradas automaticamente para mapear os atributos de entrada e saída da requisição. <br><br>
> **Observação:** Essas classes são nomeadas de acordo com o Stub dos Web Services da Senior, portanto, muitas vezes seu nome pode ser pouco conveniente ou longo demais. Estou trabalhando para melhorar isso.

Agora, basta utilizar a classe principal para acessar o Web Service G5 da Senior, como no exemplo abaixo:

```php
require_once ('./senior/services/Clientes/autoload.php');

$usuarioErp = 'teste';
$senhaErp = '123456';
$tipoCriptografia = 0;

$parametros = new clientesobterClienteIn();
$parametros->setCodigoEmpresa(1)
    ->setCodigoFilial(1)
    ->setCodigoCliente(10);

$cliente = new Clientes();
$retorno = $cliente->obterCliente($usuarioErp, $senhaErp, $tipoCriptografia, $parametros);

/* O retorno é um objeto do tipo clientesobterClienteOut conforme assinatura do método */
echo $cliente->getNomeCliente();
echo $cliente->getSaldoDuplicatas();
```

> **Observação:** Cada serviço terá em seu diretório um arquivo `autoload.php` responsável por mapear e incluir a classe principal e as auxiliares automaticamente.

### Mapeando portas especificas do WSDL

Alguns serviços possuem muitas portas diferentes ou versões de uma mesma porta. Utilizando o argumento `--operationNames` ao mapear o WSDL, você pode filtrar as portas que deseja mapear e evitar trabalho e código desnecessário.

```cmd
# php vendor/paulosscruz/senior-g5-php/bin/generator --wsdl http://example.com/g5-senior-services/sapiens_Synccom_senior_g5_co_ger_cad_clientes?wsdl --serviceName Clientes --outputDir senior/services --operationNames obterCliente
```

Para mapear mais de uma porta, basta passar uma lista com todas **separadas por uma vírgula e entre aspas**.

```cmd
# php vendor/paulosscruz/senior-g5-php/bin/generator --wsdl http://example.com/g5-senior-services/sapiens_Synccom_senior_g5_co_ger_cad_clientes?wsdl --serviceName Clientes --outputDir senior/services --operationNames "obterCliente, ExcluirClientes, GravarClientes_5"
```

### Trabalhando com *namespaces*

Para especificar o *namespace* das classes que serão geradas, você deve utilizar o argumento `--namespace` ao mapear o WSDL.

```cmd
# php vendor/paulosscruz/senior-g5-php/bin/generator --wsdl http://example.com/g5-senior-services/sapiens_Synccom_senior_g5_co_ger_cad_clientes?wsdl --serviceName Clientes --outputDir senior/services --namespace Senior\Services\Clientes
```

Agora, você pode trabalhar utilizando os [*namespaces*](https://www.php.net/manual/pt_BR/language.namespaces.php) do PHP.

```php
require_once ('./services/Clientes/autoload.php');

use Senior\Services\Clientes\Clientes;
use Senior\Services\Clientes\clientesobterClienteIn as ObterClientesIn;
use Senior\Services\Clientes\clientesobterClienteOut as ObterClientesOut;

$usuarioErp = 'teste';
$senhaErp = '123456';
$tipoCriptografia = 0;

$parametros = (new ObterClientesIn())
    ->setCodigoEmpresa(1)
    ->setCodigoFilial(1)
    ->setCodigoCliente(10);
$cliente = (new Clientes())
    ->obterCliente($usuarioErp, $senhaErp, $tipoCriptografia, $parametros);

/* O retorno é um objeto do tipo clientesobterClienteOut conforme assinatura do método */
echo $cliente->getNomeCliente();
echo $cliente->getSaldoDuplicatas();
```

> **Observação:** Considere criar namespaces indepentes para cada serviço, evitando problemas com duplicidade.


## Limitações

Por padrão a classe principal mantém o endereço do *Web Service* fixo em sua implementação, portanto, sempre que você houver alterações no endereço onde o seu servidor de aplicação (*Glassfish*) fica hospedado, você deverá mapear novamente todos os serviços utilizados.

### Alternativa

Se você reparar na implementação da classe principal, vai ver que o seu construtor possui alguns parâmetros que podem ser passados opcionalmente.

```php
/**
* @param array $options A array of config values
* @param string $wsdl The wsdl file to use
*/
public function __construct(array $options = array(), $wsdl = null) {
    //
}
```

Sugiro que você realize uma implementação adicional em sua aplicação e crie uma configuração baseada em arquivo ou classe para armazenar o endereço de todos os serviços utilizados.

Após isso, basta passar o endereço sempre que a classe principal for instanciada.

```php
require_once ('./senior/services/Clientes/autoload.php');

$usuarioErp = 'teste';
$senhaErp = '123456';
$tipoCriptografia = 0;

$parametros = new clientesobterClienteIn();
$parametros->setCodigoEmpresa(1)
    ->setCodigoFilial(1)
    ->setCodigoCliente(10);

$cliente = new Clientes([], WSDL::CLIENTES);
$retorno = $cliente->obterCliente($usuarioErp, $senhaErp, $tipoCriptografia, $parametros);

/* O retorno é um objeto do tipo clientesobterClienteOut conforme assinatura do método */
echo $cliente->getNomeCliente();
echo $cliente->getSaldoDuplicatas();
```

## Créditos

- [Paulo Cruz][link-author]
- [Todos Contribuidores][link-contributors]

[link-author]: https://github.com/paulosscruz
[link-contributors]: https://github.com/paulosscruz/senior-g5-php/graphs/contributors

## Licença

**senior-g5-php** está licenciado sob a [MIT License](http://opensource.org/licenses/MIT).