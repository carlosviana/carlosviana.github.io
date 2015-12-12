---
layout : post
title : Navegando em um Formulário WPF com o ENTER
description : "Alterando o comportamento de uma Window em WPF para navegar num formulário com a tecla ENTER, assim como o TAB."
modified : 2014-12-11
tags : [wpf, ux]
categories : [wpf]
---

Ao começar a utilizar o Windows Presentation Foundation - WPF percebi que sua estrutura é altamente personalizável, dando grande poder ao desenvolvedor sobre a interação entre o sistema e o usuário. 
Esse post trata de um dos comportamentos básicos do Windows Form, a capacidade de navegar entre os campos de um formulário através da tecla ENTER, no qual não vem habilitado por padrão no WPF.

Para ativar esse comportamento devemos sobrescrever o método **<span style="color:blue">OnStartup</span>** no arquivo *<span style="color:red">App.xaml.cs</span>*.

{% highlight csharp %}
protected override void OnStartup(StartupEventArgs e)
{
    EventManager.RegisterClassHandler(typeof(TextBox),
        TextBox.KeyUpEvent, new KeyEventHandler(TabOrderKeyUp));

    base.OnStartup(e);
}
{% endhighlight %}

Neste caso estamos adicionando no gerenciador de eventos no evento **<span style="color:green">KeyUp</span>** um delegate do método **<span style="color:blue">TabOrderKeyUp</span>**. Assim a cada evento KeyUp de um TextBox ele chamará esse método.

{% highlight csharp %}
private void TabOrderKeyUp(object sender, KeyEventArgs e)
{
    var ue = sender as FrameworkElement;
    if (e.Key == Key.Enter)
    {
        if (ue.Tag != null && ue.Tag.ToString() == "IgnoreEnterKeyTraversal")
        {
            //ignore
        }
        else
        {
            e.Handled = true;
            ue.MoveFocus(new TraversalRequest(FocusNavigationDirection.Next));
        }
    }
}
{% endhighlight %} 

Esse mótodo faz uso da classe FrameworkElement e trata o objeto enviado quando for a Key.Enter, movendo para o próximo elemento.

Caso desejemos podemos extender o registro do delegate para outros Elementos como um <span style="color:green">PasswordBox, CheckBox, ComboBox</span> entre outros.

{% highlight csharp %}
EventManager.RegisterClassHandler(typeof(CheckBox),
    CheckBox.KeyUpEvent, new KeyEventHandler(TabOrderKeyUp));

EventManager.RegisterClassHandler(typeof(ComboBox),
    ComboBox.KeyUpEvent, new KeyEventHandler(TabOrderKeyUp));

EventManager.RegisterClassHandler(typeof(PasswordBox),
    PasswordBox.KeyUpEvent, new KeyEventHandler(TabOrderKeyUp));
{% endhighlight %}