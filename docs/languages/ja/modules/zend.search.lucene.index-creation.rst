.. _zend.search.lucene.index-creation:

インデックスの構築
=========

.. _zend.search.lucene.index-creation.creating:

新しいインデックスの作成
------------

インデックスの作成機能および更新機能は、 ``Zend_Search_Lucene`` モジュールと Java Lucene
で実装されています。
これらのいずれかの機能を使用して作成したインデックスについて、
``Zend_Search_Lucene`` により検索できます。

以下の PHP コードでは、 ``Zend_Search_Lucene`` のインデックス作成 API
を用いてファイルをインデックス化する例を示します。

.. code-block:: php
   :linenos:

   // インデックスを作成します
   $index = Zend_Search_Lucene::create('/data/my-index');

   $doc = new Zend_Search_Lucene_Document();

   // ドキュメントの URL を、検索結果の ID として保存します。
   $doc->addField(Zend_Search_Lucene_Field::Text('url', $docUrl));

   // ドキュメントの内容をインデックス化します。
   $doc->addField(Zend_Search_Lucene_Field::UnStored('contents', $docContent));

   // ドキュメントをインデックスに追加します。
   $index->addDocument($doc);

新しく追加されたドキュメントは、
すぐにインデックスから取得できるようになります。

.. _zend.search.lucene.index-creation.updating:

インデックスの更新
---------

既存のインデックスを更新する際にも同じ手順を使用します。ただひとつの違いは、
create() メソッドではなく open() メソッドをコールするということです。

.. code-block:: php
   :linenos:

   // 既存のインデックスをオープンします。
   $index = Zend_Search_Lucene::open('/data/my-index');

   $doc = new Zend_Search_Lucene_Document();
   // ドキュメントの URL を、検索結果の ID として保存します。
   $doc->addField(Zend_Search_Lucene_Field::Text('url', $docUrl));
   // ドキュメントの内容をインデックス化します。
   $doc->addField(Zend_Search_Lucene_Field::UnStored('contents',
                                                     $docContent));

   // ドキュメントをインデックスに追加します。
   $index->addDocument($doc);

.. _zend.search.lucene.index-creation.document-updating:

ドキュメントの更新
---------

Lucene インデックスファイルは、ドキュメントの更新をサポートしていません。
更新するためには、いったん削除した上で改めて追加する必要があります。

そのためには、インデックス内部のドキュメント ID を使用して
``Zend_Search_Lucene::delete()`` メソッドをコールします。 この ID
は、クエリでヒットした内容から 'id' プロパティで取得できます。

.. code-block:: php
   :linenos:

   $removePath = ...;
   $hits = $index->find('path:' . $removePath);
   foreach ($hits as $hit) {
       $index->delete($hit->id);
   }

.. _zend.search.lucene.index-creation.counting:

インデックスの大きさの取得
-------------

``Zend_Search_Lucene`` のインデックスの大きさを知るには、二通りの方法があります。

``Zend_Search_Lucene::maxDoc()`` は、 最大のドキュメント番号にひとつ足した値を返します。
これは、削除されたドキュメントを含む、インデックス内のドキュメントの総数を表します。
そこで、このメソッドのシノニムとして ``Zend_Search_Lucene::count()`` を用意しました。

``Zend_Search_Lucene::numDocs()`` は、削除されていないドキュメントの総数を返します。

.. code-block:: php
   :linenos:

   $indexSize = $index->count();
   $documents = $index->numDocs();

``Zend_Search_Lucene::isDeleted($id)``
メソッドで、そのドキュメントが削除されているかどうかを調べます。

.. code-block:: php
   :linenos:

   for ($count = 0; $count < $index->maxDoc(); $count++) {
       if ($index->isDeleted($count)) {
           echo "ドキュメント #$id は削除されました。\n";
       }
   }

インデックスの最適化を行うと、削除されたドキュメントを取り除き、
ドキュメントの ID を前のほうに詰め込みます。 つまり、内部でのドキュメント ID
は変わる可能性があります。

.. _zend.search.lucene.index-creation.optimization:

インデックスの最適化
----------

Lucene のインデックスは、セグメントから構成されます。
各セグメントはデータの一部分を表し、それぞれ完全に独立しています。

Lucene インデックスセグメントのファイルは、その性質上更新することはできません。
セグメントを更新するには、セグメント全体を再構成する必要があります (Lucene
インデックスファイルのフォーマットについての詳細は、
`http://lucene.apache.org/java/2_3_0/fileformats.html`_ を参照ください) [#]_\ 。
このことより、新しいドキュメントをインデックスに追加する際には、
新しいセグメントを作成することになります。

セグメントの数が増えるとインデックスの効率が下がります。
しかし、インデックスの最適化によってこれを修復できます。
最適化により、複数のセグメントに分かれているデータがひとつにまとめられます。
この処理も、セグメントを更新することはありません。まず大きなセグメントを新しく作成し、
これまでいくつものセグメントに分かれていたデータをひとまとめにしてそこに格納し、
その後でセグメント一覧 ('segments' ファイル) を更新します。

インデックス全体の最適化を行うには、 ``Zend_Search_Lucene::optimize()``
をコールします。これは、すべてのインデックスセグメントを新しいひとつのセグメントにまとめます。

.. code-block:: php
   :linenos:

   // 既存のインデックスをオープンします
   $index = Zend_Search_Lucene::open('/data/my-index');

   // インデックスを最適化します
   $index->optimize();

自動的なインデックス最適化により、インデックスの一貫性を保ちます。

自動的な最適化は、いくつかのインデックスオプションにもとづいて段階的に進められます。
まず非常に小さなセグメントが少し大きめのセグメントに統合され、
さらにそれがもう少し大きな別のセグメントに統合され、... といった具合です。

.. _zend.search.lucene.index-creation.optimization.maxbuffereddocs:

自動最適化オプション MaxBufferedDocs
^^^^^^^^^^^^^^^^^^^^^^^^^^

**MaxBufferedDocs** は、メモリ内に溜め込まれたドキュメントを
新しいセグメントに書き出す際の最小ドキュメント数です。

**MaxBufferedDocs** の値の取得や設定は、 *$index->getMaxBufferedDocs()* あるいは
*$index->setMaxBufferedDocs($maxBufferedDocs)* のコールによって行います。

デフォルト値は 10 です。

.. _zend.search.lucene.index-creation.optimization.maxmergedocs:

自動最適化オプション MaxMergeDocs
^^^^^^^^^^^^^^^^^^^^^^^

**MaxMergeDocs** は、addDocument()
によってまとめられる最大のドキュメント数です。小さな値 (例えば 10.000 未満)
は、対話的にインデックスを作成していく際に有効です。
これにより、インデックス化の際の処理の中断時間を数秒に抑えられます。
大きな値は、バッチ処理の際に有効です。これにより、検索をより高速に行えるようになります。

**MaxMergeDocs** の値の取得や設定は、 *$index->getMaxMergeDocs()* あるいは
*$index->setMaxMergeDocs($maxMergeDocs)* のコールによって行います。

デフォルト値は PHP_INT_MAX です。

.. _zend.search.lucene.index-creation.optimization.mergefactor:

自動最適化オプション MergeFactor
^^^^^^^^^^^^^^^^^^^^^^

**MergeFactor** は、addDocument() でセグメントをまとめる頻度を指定します。
小さな値を指定すると、インデックス作成の際に使用する *RAM* の量を抑えられます。
また最適化されていないインデックスへの検索が高速になります。しかし、
インデックス作成の速度は遅くなります。大きな値を指定すると、インデックス作成の際の
*RAM*
の使用量が多くなります。また最適化されていないインデックスへの検索速度が落ちます。
しかしインデックスの作成は高速に行えます。大きな値 (> 10)
はバッチ的なインデックス作成の際に有効で、小さな値 (< 10)
は対話的なインデックス保守の際に有効です。

**MergeFactor** は、自動最適化が行われる平均セグメント数にほぼ等しくなります。
あまり大きな値を指定すると、新しいセグメントにまとめる前に
セグメント数が多くなってしまいます。これは "failed to open stream: Too many open files"
というエラーの原因となります。制限は、システムに依存します。

**MergeFactor** の値の取得や設定は、 *$index->getMergeFactor()* あるいは
*$index->setMergeFactor($mergeFactor)* のコールによって行います。

デフォルト値は 10 です。

Lucene Java および Luke (Lucene Index Toolbox -`http://www.getopt.org/luke/`_)
を使用してインデックスを最適化することもできます。 Luke の最新リリース (v0.8) は
Lucene v2.3 をベースにしており、 現在の ``Zend_Search_Lucene`` コンポーネントの実装 (Zend
Framework 1.6) と互換性があります。 古いのバージョンの ``Zend_Search_Lucene``
の実装を使う場合は、 それと互換性のある別のバージョンの Java Lucene
ツールを使う必要があります。

   - Zend Framework 1.5 - Java Lucene 2.1 (Luke tool v0.7.1 -`http://www.getopt.org/luke/luke-0.7.1/`_)

   - Zend Framework 1.0 - Java Lucene 1.4 - 2.1 (Luke tool v0.6 -`http://www.getopt.org/luke/luke-0.6/`_)



.. _zend.search.lucene.index-creation.permissions:

パーミッション
-------

インデックスファイルは、デフォルトでは全員が読み書き可能となっています。

この設定を上書きするには
``Zend_Search_Lucene_Storage_Directory_Filesystem::setDefaultFilePermissions()`` メソッドを使用します。

.. code-block:: php
   :linenos:

   // 現在のデフォルトのファイルパーミッションを取得します
   $currentPermissions =
       Zend_Search_Lucene_Storage_Directory_Filesystem::getDefaultFilePermissions();

   // 現在のユーザとグループに対してのみ読み書きアクセス権限を付与します
   Zend_Search_Lucene_Storage_Directory_Filesystem::setDefaultFilePermissions(0660);

.. _zend.search.lucene.index-creation.limitations:

制限事項
----

.. _zend.search.lucene.index-creation.limitations.index-size:

インデックスの大きさ
^^^^^^^^^^

インデックスの大きさは、 32 ビットプラットフォームでは最大 2GB までとなります。

64 ビットプラットフォームを使用すれば、 もっと大きなインデックスを扱えます。

.. _zend.search.lucene.index-creation.limitations.filesystems:

サポートするファイルシステム
^^^^^^^^^^^^^^

``Zend_Search_Lucene`` は、
検索処理やインデックス更新、インデックスの最適化を処理する際に ``flock()``
を使用しています。

PHP の `マニュアル`_ によると、 "``flock()`` は NFS
及び他の多くのネットワークファイルシステムでは動作しません" とのことです。

ネットワークファイルシステムは、 ``Zend_Search_Lucene`` では使用しないでください。



.. _`http://lucene.apache.org/java/2_3_0/fileformats.html`: http://lucene.apache.org/java/2_3_0/fileformats.html
.. _`http://www.getopt.org/luke/`: http://www.getopt.org/luke/
.. _`http://www.getopt.org/luke/luke-0.7.1/`: http://www.getopt.org/luke/luke-0.7.1/
.. _`http://www.getopt.org/luke/luke-0.6/`: http://www.getopt.org/luke/luke-0.6/
.. _`マニュアル`: http://www.php.net/manual/ja/function.flock.php

.. [#] 現在サポートしている Lucene インデックスファイルフォーマットのバージョンは
       v2.3 (Zend Framework 1.6 以降) です