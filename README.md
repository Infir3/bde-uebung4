# bde-uebung4
Indexierung mit Apache Solr

Config-Dateien für die Collection erstellen:  
```solrctl instancedir --generate $HOME/solr_configs```

Das soeben generierte Instanzverzeichnis nach ZooKeeper hochladen:  
```solrctl instancedir --create collection1 $HOME/solr_configs```  
```Uploading configs from /home/cloudera/solr_configs/conf to quickstart.cloudera:2181/solr. This may take up to a minute.```

Zweite Collection für bessere Skalierung erstellen:  
```solrctl collection --create test -s 1```

Fehlermeldung, weil es die Config bzw. Collection *test* noch nicht gibt:  
 ```<?xml version="1.0" encoding="UTF-8"?> <response> <lst name="responseHeader"> <int name="status"> 0</int> <int name="QTime"> 17033</int> </lst> <lst name="failure"> <str> org.apache.solr.client.solrj.impl.HttpSolrServer$RemoteSolrException:Error CREATEing SolrCore 'test_shard1_replica1': Unable to create core [test_shard1_replica1] Caused by: Could not find configName for collection test found:[managedTemplateSecure, predefinedTemplate, collection1, schemalessTemplate, schemalessTemplateSecure, predefinedTemplateSecure, managedTemplate]</str> </lst> </response>```

Diesmal mit dem richtigen Namen der Collection:  
 ```solrctl collection --create collection1 -s 1```  
Der Parameter -r (für replicationFactor) wird weggelassen, weil wir nur eine einzige Node in unserer VM haben.

Ordner *books* mit den Gutenberg-Beispielen indexieren:  
 ```
cd /home/cloudera/books/
java -Durl=http://$SOLRHOST:8983/solr/collection1/update -jar post.jar *.txt
Error: Unable to access jarfile post.jar
java -Durl=quickstart.cloudera:8983/solr/collection/update -jar post.jar *.txt
Error: Unable to access jarfile post.jar
```

Aus Gründen, die ich nicht verstehe, können die .txt-Dateien nicht indexiert werden. Jetzt probiere ich das gleiche mit den Beispieldateien von Cloudera (xml) nochmal:
```
cd /usr/share/doc/solr-doc*/example/exampledocs
java -Durl=http://$SOLRHOST:8983/solr/collection1/update -jar post.jar *.xml
SimplePostTool version 1.5
Posting files to base url http://:8983/solr/collection1/update using content-type application/xml..
POSTing file gb18030-example.xml
POSTing file hd.xml
POSTing file ipod_other.xml
POSTing file ipod_video.xml
POSTing file manufacturers.xml
POSTing file mem.xml
POSTing file money.xml
POSTing file monitor2.xml
POSTing file monitor.xml
POSTing file mp500.xml
POSTing file sd500.xml
POSTing file solr.xml
POSTing file utf8-example.xml
POSTing file vidcard.xml
14 files indexed.
COMMITting Solr index changes to http://:8983/solr/collection1/update..
Time spent: 0:00:00.814
```

Klappt auf Anhieb, ich stelle fest, dass es im Order *exampledocs* eine *post.jar* gibt:
```
[cloudera@quickstart exampledocs]$ ls
books.csv            ipod_video.xml     monitor.xml  solr-word.pdf
books.json           manufacturers.xml  mp500.xml    solr.xml
gb18030-example.xml  mem.xml            post.jar     test_utf8.sh
hd.xml               money.xml          post.sh      utf8-example.xml
ipod_other.xml       monitor2.xml       sd500.xml    vidcard.xml
```

Schieben wir die *post.jar* und andere Dateien, die vllt. helfen könnten, mal ins Book-Verzeichnis und gucken, was dann passiert:
```
cp post.jar /home/cloudera/books
cp post.sh /home/cloudera/books
cp test_utf8.sh /home/cloudera/books
SimplePostTool: WARNING: IOException while reading response: java.io.IOException: Server returned HTTP response code: 400 for URL: http://:8983/solr/collection1/update
POSTing file pg22657.txt
SimplePostTool: WARNING: Solr returned an error #400 (Bad Request) for url: http://:8983/solr/collection1/update
SimplePostTool: WARNING: Response: <?xml version="1.0" encoding="UTF-8"?>
<response>
<lst name="responseHeader"><int name="status">400</int><int name="QTime">1</int></lst><lst name="error"><lst name="metadata"><str name="error-class">org.apache.solr.common.SolrException</str><str name="root-error-class">com.ctc.wstx.exc.WstxUnexpectedCharException</str></lst><str name="msg">Unexpected character 'P' (code 80) in prolog; expected '&lt;'
 at [row,col {unknown-source}]: [1,1]</str><int name="code">400</int></lst>
</response>
SimplePostTool: WARNING: IOException while reading response: java.io.IOException: Server returned HTTP response code: 400 for URL: http://:8983/solr/collection1/update
POSTing file pg2591.txt
SimplePostTool: WARNING: Solr returned an error #400 (Bad Request) for url: http://:8983/solr/collection1/update

[...]

SimplePostTool: WARNING: Response: <?xml version="1.0" encoding="UTF-8"?>
<response>
<lst name="responseHeader"><int name="status">400</int><int name="QTime">1</int></lst><lst name="error"><lst name="metadata"><str name="error-class">org.apache.solr.common.SolrException</str><str name="root-error-class">com.ctc.wstx.exc.WstxUnexpectedCharException</str></lst><str name="msg">Unexpected character 'T' (code 84) in prolog; expected '&lt;'
 at [row,col {unknown-source}]: [3,1]</str><int name="code">400</int></lst>
</response>
SimplePostTool: WARNING: IOException while reading response: java.io.IOException: Server returned HTTP response code: 400 for URL: http://:8983/solr/collection1/update
20 files indexed.
COMMITting Solr index changes to http://:8983/solr/collection1/update..
Time spent: 0:00:00.536
[cloudera@quickstart books]$
```

Zumindest passiert jetzt was, allerdings gibt es nun eine andere Fehlermeldung. Zumindest das Indexieren der Beispieldateien hat geklappt. Ich vermute, dass noch irgendwas an der Konfiguration geändert werden muss, damit auch die .txt-Dateien richtig verarbeitet werden können.
