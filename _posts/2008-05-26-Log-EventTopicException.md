---
layout: post
title: Logger les exceptions de type EventTopicException
tags:
 - SCSF
 - Enterprise Library
---

Lorsque l'on utilise le SCSF/CAB, il arrive parfois que l'on se retrouve confronté à des exceptions quelques peux obscure du genre 'One or more exceptions occurred while firing the topic ....' .

Ces exceptions sont levées lorsqu'une fonction appellée dans un Event CAB à un problème.
Pour avoir plus d'informations lors du debugging, on peux simplement examiner la propriété Exceptions qui contient un tableau des exceptions levée lors de l'appel à l'événement.
Mais comment faire lorsque l'application n'est plus sous contrôle du développeur et que le log vous renvoie uniquement ce magnifique message 'One or more blah blah...' ??

Eh bien en utilisant la classe EventTopicExceptionFormatter qui nous est créée automatiquement par SCSF dans Infrastructure.Library. Il suffit ensuite de configurer un nouveau type d'exception dans EntLib:

{% highlight xml %}
<exceptionHandling>
  <exceptionPolicies>
    <add name="Default Policy">
    <exceptionTypes>
      <add type="Microsoft.Practices.CompositeUI.EventBroker.EventTopicException, Microsoft.Practices.CompositeUI, Version=1.0.51205.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"
          postHandlingAction="NotifyRethrow" name="EventTopicException">          
          <exceptionHandlers>            
            <add logCategory="General" eventId="100" severity="Error" title="GMS Exception Handling"
                 formatterType="YourNamespace.Infrastructure.Library.EntLib.EventTopicExceptionFormatter, Infrastructure.Library"              
                 priority="0" type="Microsoft.Practices.EnterpriseLibrary.ExceptionHandling.Logging.LoggingExceptionHandler, Microsoft.Practices.EnterpriseLibrary.ExceptionHandling.Logging, Version=3.1.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"              
                 name="Logging Handler" />          
          </exceptionHandlers>
      </add>
    </exceptionTypes>
    </add>  
  </exceptionPolicies>
 </exceptionHandling>
{% endhighlight %}

Et voilà! Maintenant vos logs contienent toutes les exceptions levées lors de votre event.
