.. _zend.json.basics:

Basisgebruik
============

Het gebruik van *Zend_Json* impliceert het gebruik van de volgende twee publiek beschikbare methodes:
*Zend_Json::encode()* en *Zend_Json::decode()*.

   .. code-block::
      :linenos:
      <?php
      // Een waarde verkrijgen:
      $phpNative = Zend_Json::decode($encodedValue);

      // Encodering om het terug naar de klant te sturen:
      $json = Zend_Json::encode($phpNative);
      ?>



