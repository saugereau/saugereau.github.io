---
layout: post
title: Solr stopwords on multiple field
category: Architecture
tags: solr search
summary: When you use solr, and query on multiple field, you could have some trouble ..
---

I recently encountered a problem on Solr 4.3.1, I can reproduce it based upon the sample collection embedded on the zip archive.
For your sample, you could download the Solr 4.3.1 version (in archive section), unzip it.

### Prepare the Solr intance

This distribution comes with a collection1 folder in \solr-4.3.1\example\solr. You could copy/paste this folder and rename in collection2 for instance an do some updates :

- Remove the directories index and tlog in data folder
- Edit the solr.xml to add the collection2

```xml

 <cores adminPath="/admin/cores" defaultCoreName="collection1" host="${host:}" hostPort="${jetty.port:8983}" hostContext="${hostContext:solr}" zkClientTimeout="${zkClientTimeout:15000}">
    <core name="collection1" instanceDir="collection1" />
	<core name="collection2" instanceDir="collection2" />
  </cores>

```

- Update the schema.xml file of your new collection

```xml

<fields>

    <field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" />

    <field name="sku" type="string" indexed="true" stored="true" omitNorms="true"/>
    <field name="barcode" type="string" indexed="true" stored="true" omitNorms="true"/>
    <field name="categoryid" type="long" indexed="true" stored="true"/>
    <field name="isbn" type="text_en_splitting_tight" indexed="true" stored="true" omitNorms="true"/>
    <field name="name" type="text_fr" indexed="true" stored="true"/>
    <field name="description" type="text_fr" indexed="true" stored="true"/>

    <field name="ids" type="text_SF" indexed="true" stored="false" multiValued="true"/>

    <field name="_version_" type="long" indexed="true" stored="true"/>
</fields>

<uniqueKey>id</uniqueKey>

<copyField source="sku" dest="ids"/>
<copyField source="barcode" dest="ids"/>
<copyField source="isbn" dest="ids"/>


```

- Update the solrconfig.xml file

```xml

<requestHandler name="/select" class="solr.SearchHandler">
    <!-- default values for query parameters can be specified, these
         will be overridden by parameters in the request
      -->
     <lst name="defaults">
	   <str name="defType">dismax</str>
       <str name="echoParams">explicit</str>
       <int name="rows">10</int>
	   <str name="q.op">AND</str>
       <str name="qf">ids name description^0.5</str>
     </lst>
</requestHandler>

```

### Put the datas

I  create a simple xml file to load my datas :

```xml
<add>
	<doc>
		<field name="id">3144876</field>
		<field name="barcode">9782703008484</field>
		<field name="categoryid">1000015310</field>
		<field name="name">Les cigognes</field>
		<field name="description">
De Fischer Nagel paru le 01 janvier 2070 aux éditions GAMMA EDITIONS
		</field>
		<field name="isbn">2703008481</field>
		<field name="sku">9782703008484</field>
	</doc>
	<doc>
		<field name="id">1011301</field>
		<field name="barcode">9782867269509</field>
		<field name="categoryid">1000015310</field>
		<field name="name">La cigogne</field>
		<field name="description">
De Christine Denis-Huot paru le 14 mars 1994 aux éditions MILAN
		</field>
		<field name="isbn">2867269504</field>
		<field name="sku">9782867269509</field>
	</doc>
	<doc>
		<field name="id">10858951</field>
		<field name="barcode">9791023205008</field>
		<field name="categoryid">1000015310</field>
		<field name="name">LES MOUSQUETAIRES</field>
		<field name="description">LES MOUSQUETAIRES : LA TRILOGIE - ALEXANDRE DUMAS</field>
		<field name="sku">9791023205008</field>
	</doc>
	<doc>
		<field name="id">3599285</field>
		<field name="barcode">9782895371786</field>
		<field name="categoryid">1000015310</field>
		<field name="name">Eulalie la cigogne</field>
		<field name="description">
De Kaye Veronique-Marie paru le 16 février 2010 aux éditions VENTS D'OUEST CANADA
		</field>
		<field name="isbn">2895371784</field>
		<field name="sku">9782895371786</field>
	</doc>
	<doc>
		<field name="id">5565194</field>
		<field name="barcode">9780307779878</field>
		<field name="categoryid">1000015310</field>
		<field name="name">ROBIN HOOD ROBIN DES BOIS</field>
		<field name="description">ROBIN HOOD ROBIN DES BOIS - COLLECTIF</field>
		<field name="sku">9780307779878</field>
	</doc>
	<doc>
		<field name="id">311237</field>
		<field name="barcode">9782226119063</field>
		<field name="categoryid">1000015310</field>
		<field name="name">Le chaperon rouge</field>
		<field name="description">
De Charles Perrault - Joelle Jolivet paru le 27 octobre 2002 aux éditions ALBIN MICHEL JEUNESSE
		</field>
		<field name="isbn">222611906X</field>
		<field name="sku">9782226119063</field>
	</doc>
	<doc>
		<field name="id">545711</field>
		<field name="barcode">9782020323499</field>
		<field name="categoryid">1000015310</field>
		<field name="name">MON CHAPERON ROUGE</field>
		<field name="description">
De Anne Ikhlef - Alain Gauthier paru le 02 septembre 1998 aux éditions SEUIL
		</field>
		<field name="isbn">2020323494</field>
		<field name="sku">9782020323499</field>
	</doc>
	<doc>
		<field name="id">3226490</field>
		<field name="barcode">9782803413935</field>
		<field name="categoryid">1000015310</field>
		<field name="name">CHAPERON ROUGE</field>
		<field name="description">
De Collectif paru le 26 avril 1998 aux éditions CHANTECLER
		</field>
		<field name="sku">9782803413935</field>
	</doc>
</add>

```

To import data, I use the post.jar (in solr-4.3.1\example\exampledocs), to see post option you could use :

```
java -jar post.jar -help
```

So to import xml file to specific collection

```
java -Durl=http://localhost:8983/solr/collection2/update -jar post.jar tenproducts.xml
```

### Test it

So if you do a search on "cigogne", you get the document with id 3144876, 1011301 and 3599285, however if you do the search with "les cigognes",
there's nothing returned instead while there is a document with name "Les cigognes".

Name and description are using stop filter on index and query analyser, an example of "les cigognes" both index and query analyser :

<figure>
  <img src="/blog/assets/images/solr-dismax-stopwords/query_analysis.png" />
  <figcaption>Solr query analysis screen</figcaption>
</figure>

It seems that this filter doesn't work any more when we query.
A simple try removing ids from /select qf :

```xml

<requestHandler name="/select" class="solr.SearchHandler">
    <!-- default values for query parameters can be specified, these
         will be overridden by parameters in the request
      -->
     <lst name="defaults">
	   <str name="defType">dismax</str>
       <str name="echoParams">explicit</str>
       <int name="rows">10</int>
	   <str name="q.op">AND</str>
       <str name="qf">name description^0.5</str>
     </lst>
</requestHandler>

```

It works, the more simple hack seems to have the same type of field on the list in qf field, or with the same stop word filter.

```xml

<field name="ids" type="text_fr" indexed="true" stored="false" multiValued="true"/>

```


This problem also exist on Solr 4.10.1, and I think that's in relationship with this [bug](https://issues.apache.org/jira/browse/SOLR-3085)