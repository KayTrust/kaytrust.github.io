# Productos

El activo KayTrust cuenta con los siguientes productos:

* Protocolos de Identidad
* Identidad Digital
* SDK Hub
* SDK Provider
* KayTrust Hub
* KayTrust Provider
* KayTrust Wallet


## Protocolos de Identidad

* Proxy Contract
  ....
  
* Proxy Type
  ....


## Identidad Digital 

KayTrust Digital Identity - Proveedor de Identidad Digital   

	..........
	..........
	

Digital Identity - Dependencias

    Usando MAVEN
	
		<dependency>
			<groupId>com.everis.blockchain</groupId>
			<artifactId>algorithm</artifactId>
			<version>0.0.4</version>
		</dependency>
   
    O descargandote las librerias de la siguiente URL : [AzureRepos.id](https://azurerepos.id/).

Digital Identity - Clases

    * Crear Credenciales ... ..
	
	* Crear Contrato
	
	* ....

Digital Identity - Demo Crear Credenciales

	* Importar librerias
	
	package demo.acme.kaytrust
	
	import com.everis.blockchain.algorithm.web3j.html.Account;
    import com.everis.blockchain.algorithm.web3j.html.EthCore;
    import com.everis.blockchain.algorithm.web3j.html.EthCoreParams;

    public class GeneratePrivateKey {
	
	    private static final Logger log = LoggerFactory.getLogger("GeneratePrivateKey");
	  
	  	public void create() throws Exception {

			Account account = ethCore.createCredentials("");

			log.info("PRIVATE_KEY_BACKEND : " + account.getPrivateKey());
			log.info("ADDRESS_ETHEREUM_BACKEND : " + account.getAddress());
		}

	}



## SDK Hub

    In construction ..

## SDK Provider

    In construction ..

## KayTrust Hub

    In construction ..
	
## KayTrust Provider

    In construction ..

## KayTrust Wallet

    In construction ..
