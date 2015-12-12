---
layout : post
title : Aplicando Listeners em camadas na arquitetura MVVM
description : "Criando um serviço de Listener que permite enviar uma mensagem de fora da camada da ViewModel diretamente a View."
modified : 2015-12-11
tags : [wpf, helpers]
categories : [wpf]
---

Nesse post vou abstrair informações conceituais da *<span style="color:red">Arquitetura MVVM</span>* caso tenha dúvidas esses links podem lhe ajudar:

- [MVVM pelo CodeProject](http://www.codeproject.com/Articles/100175/Model-View-ViewModel-MVVM-Explained)


Basicamente, entre outras características a Arquitetura MVVM permite isolar a lógica de controle de comportamentos de uma tela em uma camada chamada **<span style="color:blue">ViewModel</span>**. 
A ViewModel utiliza-se do recurso de *<span style="color:red">DataBinding</span>* extremamente poderoso do ambiente WPF.

Porém haverá situações específicas que outras camadas abaixo da **<span style="color:blue">ViewModel</span>** tenham a necessidade de notificar visualmente a **<span style="color:blue">View</span>**. 

Alguns casos que podemos aplicar esse conceito:

* Mensagens de Status de acompanhamento de processos da camada de regras de negócio;
* Notificações por ProgressBars;

Para fins de exemplificar em um ambiente. Teremos uma View *<span style="color:blue">WindowImport.xaml</span>* que mostra ao usuário os status a medida que um arquivo é importado.
A Viewmodel *<span style="color:blue">ImportViewModel</span>* realiza o controle e a interação das camadas de Modelo e Controle com a View.
O que queremos é que a View mostre o Status vinda de uma classe de Controle.

Na classe View  utilizaremos um **<span style="color:red">TextBlock</span>** para informar o status.

{% highlight csharp %}
<TextBlock Name="tbMessageStatus" Text="{Binding Source={x:Static serv:StatusListener.Instance}, Path=Message}"</TextBlock>
{% endhighlight %}

Na propriedade *Text* será feito um binding a uma instancia estática da classe de listener *<span style="color:red">StatusListener</span>* 

Para acessar através da View esta classe estática devemos incluír uma chamada no seu namespace.

{% highlight csharp %}
xmlns:serv="clr-namespace:Solution.Services"
{% endhighlight %}

Aqui temos a classe *<span style="color:red">StatusListener.cs</span>*

{% highlight csharp %}
public class StatusListener : DependencyObject
{
	private static StatusListener mInstance;
	       

	public static StatusListener Instance
	{
	    get
	    {
	        if (mInstance == null)
	            mInstance = new StatusListener();

	        return mInstance;
	    }
	}

	public void RecieveMessage(string message)
	{
	    Message = message;
	    DispatcherHelper.DoEvents();
	}

	public string Message
	{
	    get { return (string) GetValue(MessageProperty); }
	    set { SetValue(MessageProperty, value); }
	}

	public static readonly DependencyProperty MessageProperty =
	    DependencyProperty.Register("Message", typeof(string), typeof(StatusListener), new UIPropertyMetadata(null));

}
{% endhighlight %}

Para disparar a mensagem no método **<span style="color:blue">RecieveMessage</span>** será necessário criar e chamar o método *DoEvents()* da classe **DispatcherHelper**.

{% highlight csharp %}
public static class DispatcherHelper
{   
    [SecurityPermissionAttribute(SecurityAction.Demand, Flags = SecurityPermissionFlag.UnmanagedCode)]
    public static void DoEvents()
    {
        DispatcherFrame frame = new DispatcherFrame();
        Dispatcher.CurrentDispatcher.BeginInvoke(DispatcherPriority.Background,
            new DispatcherOperationCallback(ExitFrames), frame);

        try
        {
            Dispatcher.PushFrame(frame);
        }
        catch (InvalidOperationException)
        {
        }
    }


    private static object ExitFrames(object frame)
    {
        ((DispatcherFrame)frame).Continue = false;

        return null;
    }
}
{% endhighlight %}


Agora para finalizar falta somente chamar o serviço de **Listener** para disparar uma mensagem diretamente para a View.

{% highlight csharp %}
StatusListener.Instance.RecieveMessage(string.Format("Iniciando importação..."));
{% endhighlight %}

Vale salientar que não há ha necessidade de instanciar o **Listener** devido a ele ser estático.

O método de RecieveMessage recebe uma string, porém podemos utilizar outros tipos como int (muito utilizado em ProgessBars) ou boolean. 