
# Aplicação Cliente Servidor J2EE - Interface Remota - IIOP

![](https://lh3.googleusercontent.com/-m7XWvK0733w/VDnnjOkrXdI/AAAAAAAAB40/WFqGRPEiuVA/w888-h308-no/ClienteServidor.jpeg)

Este projeto é uma tradução para o português do projeto https://github.com/wildfly/quickstart/tree/master/ejb-remote

## Lado Servidor 
![](https://lh5.googleusercontent.com/-z5nHS1CzypA/VDnqPPlDknI/AAAAAAAAB5Q/CtA9Vjy51Q8/w370-h306-no/Servidor.jpg)

O programa servidor irá ser carregado em um ambiente Servidor J2EE versão 7 ( container ) com suporte a ivocação de métodos remotos ativado.
Escolhi, por conveniência, o Red Hat WildFly 8.1 final.

### Exemplo de EJB Sem Estado Remoto
###Dependências:
Caso você esteja usando o maven, para este projeto, teremos as seguintes dependências:

A biblioteca **jboss-annotations-api_1.2_spec**
para fazer esta importação:
```java
import javax.annotation
```
A biblioteca **jboss-ejb-api_3.2_spec**
para fazer esta importação:
```java
import javax.ejb
```


***
Interface CalculadoraRemota.java
```java
package miguel.servidor.remoto.sem_estado;

public interface CalculadoraRemota
{
	
	int soma(int parcela1,int parcela2);
		
	int subtracao(int parcela1,int parcela2);
	
}

```
Classe Calculadora.java
```java
package miguel.servidor.remoto.sem_estado;

import javax.ejb.Remote;
import javax.ejb.Stateless;

@Stateless
@Remote(CalculadoraRemota.class)
public class Calculadora implements CalculadoraRemota
{
	@Override
	public int soma(int parcela1,int parcela2)
	{
		int resultado;
		resultado = (parcela1 + parcela2);
		return(resultado);
	};
	
	@Override
	public int subtracao(int parcela1,int parcela2)
	{
		int resultado;
		resultado = (parcela1 - parcela2);
		return(resultado);
	};
}

```

### Exemplo de EJB Com Estado Remoto
Interface ContadorRemoto.java
```java
package miguel.servidor.remoto.com_estado;

public interface ContadorRemoto
{
    void incrementa();
    
    void decrementa();
 
    int mostra_contagem();
}

```
Classe Contador.java
```java
package miguel.servidor.remoto.com_estado;


import javax.ejb.Remote;
import javax.ejb.Stateful;


@Stateful
@Remote(ContadorRemoto.class)
public class Contador implements ContadorRemoto
{
	private int contagem =0;

	@Override
	public void incrementa()
	{
		this.contagem++;

	}

	@Override
	public void decrementa()
	{
		this.contagem--;

	}

	@Override
	public int mostra_contagem()
	{
		return this.contagem;
	}

}

```

***
## Lado Cliente
![](https://lh6.googleusercontent.com/-wosJZGncLXA/VDnqPNDSmBI/AAAAAAAAB5M/WR3aibfbdeE/w370-h308-no/Cliente.jpg)

Classe ClienteRemoto.java
```java
package miguel.cliente.remoto;

import javax.naming.Context;
import javax.naming.InitialContext;
import javax.naming.NamingException;
import java.security.Security;
import java.util.Hashtable;

import miguel.servidor.remoto.sem_estado.CalculadoraRemota;
import miguel.servidor.remoto.sem_estado.Calculadora;
import miguel.servidor.remoto.com_estado.ContadorRemoto;
import miguel.servidor.remoto.com_estado.Contador;

import org.jboss.sasl.JBossSaslProvider;


public class ClienteEJBRemoto
{

	public static void main(String[] args) throws Exception
	{
		 // Invoke a stateless bean
        invocarCalculadoraRemota();
 
        // Invoke a stateful bean
        invocarContadorRemoto();

	}

	
    private static void invocarCalculadoraRemota() throws NamingException 
    {
        
        final CalculadoraRemota objetoCalculadoraRemota = buscaCalculadoraRemotaServidor();
        System.out.println("Objeto Calculadora invocado e instanciado.");
        
        int a = 204;
        int b = 340;
        System.out.println("Somando " + a + " e " + b + " no servidor");
        int resultado = objetoCalculadoraRemota.soma(a, b);
        System.out.println("Retornada a soma = " + resultado);
        if (resultado != a + b) 
        {
            throw new RuntimeException("A calculadora remota retornou soma incorreta " + resultado + " ,o esperado era: " + (a + b));
        }
        
        
        int num1 = 3434;
        int num2 = 2332;
        System.out.println("Subtraindo " + num2 + " de " + num1 + " no servidor.");
        int diferenca = objetoCalculadoraRemota.subtracao(num1, num2);
        System.out.println("Calculadora remota retornou a diferenca = " + diferenca);
        if (diferenca != num1 - num2) 
        {
            throw new RuntimeException("A calculadora remota retornou soma incorreta " + diferenca + " ,o esperado era: " + (num1 - num2));
        }
    }
    
    private static CalculadoraRemota buscaCalculadoraRemotaServidor() throws NamingException 
    {
        final Hashtable propriedadesJNDI = new Hashtable();
        propriedadesJNDI.put(Context.URL_PKG_PREFIXES, "org.jboss.ejb.client.naming");
        final Context contexto = new InitialContext(propriedadesJNDI);
 
        final String nomeDaAplicacao = "";
  
        final String nomeDoModulo = "jboss-as-ejb-remote-app";
  
        final String distinctName = "";
 
        final String nomeDaClasse = Calculadora.class.getSimpleName();
 
        final String nomeDaInterface = CalculadoraRemota.class.getName();
  
        return (CalculadoraRemota) contexto.lookup("ejb:" + nomeDaAplicacao + "/" + nomeDoModulo + "/" + distinctName + "/" + nomeDaClasse + "!" + nomeDaInterface);
    }
 	
    private static void invocarContadorRemoto() throws NamingException 
    {
        
        final ContadorRemoto objetoContadorRemoto = buscaContadorRemotoServidor();
        System.out.println("Invocado o contador remoto.");
        
        final int NUMERO_DE_VEZES = 20;
        System.out.println("Contador será agora incrementado " + NUMERO_DE_VEZES + " vezes");
        for (int i = 0; i < NUMERO_DE_VEZES; i++) {
            System.out.println("Incrementando contador...");
            objetoContadorRemoto.incrementa();
            System.out.println("contador depois do incremento " + objetoContadorRemoto.mostra_contagem());
        }
        
        System.out.println("Contador será agora decrementado " + NUMERO_DE_VEZES + " vezes");
        for (int i = NUMERO_DE_VEZES; i > 0; i--) {
            System.out.println("Decrementando o contador...");
            objetoContadorRemoto.decrement();
            System.out.println("Contador depois do decremento " + objetoContadorRemoto.mostra_contagem());
        }
    }

    private static ContadorRemoto buscaContadorRemotoServidor() throws NamingException {
        final Hashtable propriedadesJNDI = new Hashtable();
        propriedadesJNDI.put(Context.URL_PKG_PREFIXES, "org.jboss.ejb.client.naming");
        final Context contexto = new InitialContext(propriedadesJNDI);
 
        final String nomeDaAplicacao = "";
 
        final String nomeDoModulo = "jboss-as-ejb-remote-app";
 
        final String distinctName = "";
        
        final String nomeDaClasse = Contador.class.getSimpleName();
        
        final String nomeDaInterface = ContadorRemoto.class.getName();
        
        return (ContadorRemoto) contexto.lookup("ejb:" + nomeDaAplicacao + "/" + nomeDoModulo + "/" + distinctName + "/" + nomeDaClasse + "!" + nomeDaInterface + "?stateful");
    }
    
}

```

Propriedades de EJB no WildFly 8.0

Arquivo jboss-ejb-client.properties

```xml

endpoint.name=client-endpoint
remote.connectionprovider.create.options.org.xnio.Options.SSL_ENABLED=false
remote.connections=default
 
remote.connection.default.host=localhost
remote.connection.default.port = 4447
remote.connection.default.connect.options.org.xnio.Options.SASL_POLICY_NOANONYMOUS=false

remote.connection.default.username=conectajava
remote.connection.default.password=dGVzdGUxMjM=

```
