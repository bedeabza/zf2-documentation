.. _zend.amf.introduction:

Introdução
==========

``Zend_Amf`` fornece suporte para o `Action Message Format`_ (*AMF*) da Adobe, para permitir a comunicação entre
o `Adobe Flash Player`_ e o *PHP*. Especificamente, ele fornece uma implementação do servidor de gateway para
tratar as solicitações enviadas a partir do Flash Player para o servidor e mapeá-las para os objetos e métodos
de classe e callbacks arbitrários.

A `especificação AMF3`_ está disponível livremente, e serve como referência para os tipos de mensagens que
podem ser enviadas entre o Flash Player e o servidor.



.. _`Action Message Format`: http://en.wikipedia.org/wiki/Action_Message_Format
.. _`Adobe Flash Player`: http://en.wikipedia.org/wiki/Adobe_Flash_Player
.. _`especificação AMF3`: http://download.macromedia.com/pub/labs/amf/amf3_spec_121207.pdf
