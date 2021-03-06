.. _zend.log.writers.mail:

In Emails schreiben
===================

``Zend_Log_Writer_Mail`` schreibt Logeinträge in eine Email Nachricht indem es ``Zend_Mail`` verwendet. Der
Konstruktor von ``Zend_Log_Writer_Mail`` nimmt ein ``Zend_Mail`` Objekt und ein optionales ``Zend_Layout`` Objekt
entgegen.

Der primäre Verwendungszweck für ``Zend_Log_Writer_Mail`` ist es Entwickler, System Administratoren, oder jede
benötigte Partei über Fehler zu benachrichtigen die mit *PHP*-basierenden Skripten auftreten können.
``Zend_Log_Writer_Mail`` wurde aus der Idee geboren das, wenn irgendetwas fehlschlägt, ein Mensch sofort darüber
alarmiert werden muß um korrigierende Maßnahmen durchzuführen.

Die grundsätzliche Verwendung wird anbei gezeigt:

.. code-block:: php
   :linenos:

   $mail = new Zend_Mail();
   $mail->setFrom('errors@example.org')
        ->addTo('project_developers@example.org');

   $writer = new Zend_Log_Writer_Mail($mail);

   // Setzt den Subjekt-Text der verwendet wird; Zusammenfassung der Anzahl der
   // Fehler wird der Subjektzeile angefügt bevor die Nachricht gesendet wird.
   $writer->setSubjectPrependText('Errors with script foo.php');

   // Nur Einträge vom Level Warnung und höher schicken
   $writer->addFilter(Zend_Log::WARN);

   $log = new Zend_Log();
   $log->addWriter($writer);

   // Irgendetwas schlechtes findet statt!
   $log->error('Kann nicht zur datenbank verbinden');

   // Wenn der Writer heruntergefahren wird, wird Zend_Mail::send()
   // getriggert um ein Email mit allen Logeinträgen zu senden die
   // auf oder über dem Filterlevel von Zend_Log liegen

``Zend_Log_Writer_Mail`` stellt den Email Body standardmäßig als reinen Text dar.

Es wird eine Email gesendet die alle Logeinträge enthält die auf oder über dem Filterlevel sind. Wenn zum
Beispiel, Einträge mit Warnungs-Level gesendet werden sollen, und zwei Warnungen und fünf Fehler stattfinden,
wird das Email in Summe sieben Logeinträge enthalten.

.. _zend.log.writers.mail.layoutusage:

Zend_Layout Verwendung
----------------------

Eine ``Zend_Layout`` Instanz kann verwendet werden um den *HTML* Anteil einer Multipart Email zu erstellen. Wenn
eine ``Zend_Layout`` Instanz in Verwendung ist, nimmt ``Zend_Log_Writer_Mail`` an das sie verwendet wird um *HTML*
darzustellen und setzt den Body *HTML* für die Nachricht als den ``Zend_Layout``-darstellenden Wert.

Wenn ``Zend_Log_Writer_Mail`` mit einer ``Zend_Layout`` Instanz verwendet wird, hat man die Option eine eigene
Formatierung zu setzen indem die ``setLayoutFormatter()`` Methode verwendet wird. Wenn kein
``Zend_Layout``-spezifischer formatierungs-Eintrag spezifiziert wurde, wird der aktuell in Verwendung befindliche
Formatierer verwendet. Die vollständige Verwendung von ``Zend_Layout`` mit einem eigenen Formatierer wird anbei
gezeigt.

.. code-block:: php
   :linenos:

   $mail = new Zend_Mail();
   $mail->setFrom('errors@example.org')
        ->addTo('project_developers@example.org');
   // Beachte das die Subjektzeile nicht auf der Zend_Mail
   // Instanz gesetzt wird!

   // Verwende eine einfache Zend_Layout Instanz mit seinen Standardwerten
   $layout = new Zend_Layout();

   // Erstelle einen Formatierer der den Eintrag in ein Listitem Tag umhüllt
   $layoutFormatter = new Zend_Log_Formatter_Simple(
       '<li>' . Zend_Log_Formatter_Simple::DEFAULT_FORMAT . '</li>'
   );

   $writer = new Zend_Log_Writer_Mail($mail, $layout);

   // Füge den Formatierer für die Einträge hinzu die mit Zend_Layout
   // dargestellt werden
   $writer->setLayoutFormatter($layoutFormatter);
   $writer->setSubjectPrependText('Fehler im Skript foo.php');
   $writer->addFilter(Zend_Log::WARN);

   $log = new Zend_Log();
   $log->addWriter($writer);

   // Irgendetwas schlechtes hat stattgefunden!
   $log->error('unable to connect to database');

   // Wenn der Writer heruntergefahren wird, wird Zend_Mail::send()
   // getriggert um ein Email mit allen Logeinträgen zu senden die
   // auf oder über dem Filterlevel von Zend_Log liegen

.. _zend.log.writers.mail.dynamicsubjectline:

Zusammenfassung der Fehlerlevel in der Subjektzeile
---------------------------------------------------

Die ``setSubjectPrependText()`` Methode kann statt ``Zend_Mail::setSubject()`` verwendet werden um die Subjektzeile
dynamisch zu schreiben bevor die Email gesendet wird. Wenn, zum Beispiel, der dem Subjekt vorangestellte Text
"Fehler des Skriptes" heißt, würde das Subjekt des Emails das von ``Zend_Log_Writer_Mail`` mit zwei Warnungen und
fünf Fehlern erstellt wird "Fehler des Skriptes (warn = 2; error = 5)" sein. Wenn kein Subjekttext mit
``Zend_Log_Writer_Mail`` vorangestellt wird, wird die ``Zend_Mail`` Subjektzeile verwendet, wenn vorhanden.

.. _zend.log.writers.mail.caveats:

Vorbehalte
----------

Das Senden von Logeinträgen über Emails kann gefährlich sein. Wenn Fehlerzustände vom eigenen Skript falsch
behandelt werden, oder man die Fehlerlevel falsch verwendet, kann man sich selbst in einer Situation finden in der
man Hunderte oder Tausende von Emails zu den Empfängern sendet abhängig von der Frequenz der eigenen Fehler.

Bis dato bietet ``Zend_Log_Writer_Mail`` keinen Mechanismus für die Limitierung oder andernfalls die Dosierung der
Nachrichten. Solche Funktionalitäten, können vom Konsumenten, wenn benötigt, selbst implementiert werden.

Nochmals, ``Zend_Log_Writer_Mail``'s primäres Ziel ist es proaktiv Menschen über Fehlerzustände zu
benachrichtigen. Wenn diese Fehler zeitgerecht behandelt werden, und Sicherheitsmaßnahmen platziert werden um
diese Vorbehalte in Zukunft zu verhindern, dann kann die Email-basierende Benachtichtigung von Fehlern ein
nützliches Tool sein.


