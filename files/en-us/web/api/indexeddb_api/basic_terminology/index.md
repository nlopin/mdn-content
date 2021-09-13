---
title: IndexedDB key characteristics and basic terminology
slug: Web/API/IndexedDB_API/Basic_Terminology
tags:
  - Advanced
  - IndexedDB
  - terminology
---
<p>{{DefaultAPISidebar("IndexedDB")}}</p>

<p>This article describes the key characteristics of IndexedDB, and introduces some essential terminology relevant to understanding the IndexedDB API.</p>

<p>You'll also find the following articles useful:</p>

<ul>
 <li>For a detailed tutorial on how to use the API, see <a href="/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB">Using IndexedDB</a>.</li>
 <li>For the reference documentation on the IndexedDB API, refer back to the main <a href="/en-US/docs/Web/API/IndexedDB_API">IndexedDB API</a> article and its subpages, which document the types of objects used by IndexedDB.</li>
 <li>For more information on how the browser handles storing your data in the background, read <a href="/en-US/docs/Web/API/IndexedDB_API/Browser_storage_limits_and_eviction_criteria">Browser storage limits and eviction criteria</a>.</li>
</ul>

<h2 id="key_characteristics">Key characteristics</h2>

<p>IndexedDB is a way for you to persistently store data inside a user's browser. Because it lets you create web applications with rich query abilities regardless of network availability, these applications can work both online and offline. IndexedDB is useful for applications that store a large amount of data (for example, a catalog of DVDs in a lending library) and applications that don't need persistent internet connectivity to work (for example, mail clients, to-do lists, and notepads).</p>

<p>IndexedDB lets you store and retrieve objects that are indexed with a "key." All changes that you make to the database happen within transactions. Like most web storage solutions, IndexedDB follows a <a href="https://www.w3.org/Security/wiki/Same_Origin_Policy">same-origin policy</a>. So while you can access stored data within a domain, you cannot access data across different domains.</p>

<p>If you have assumptions from working with other types of databases, you might get thrown off when working with IndexedDB. So the following key characteristics of IndexedDB are important to keep in mind:</p>

<ul>
 <li>
  <p><strong>IndexedDB databases store key-value pairs.</strong> The values can be complex structured objects, and keys can be properties of those objects. You can create indexes that use any property of the objects for quick searching, as well as sorted enumeration. Keys can be binary objects.</p>
 </li>
 <li>
  <p><strong>IndexedDB is built on a transactional database model</strong>. Everything you do in IndexedDB always happens in the context of a <a href="#transaction">transaction</a>. The IndexedDB API provides lots of objects that represent indexes, tables, cursors, and so on, but each of these is tied to a particular transaction. Thus, you cannot execute commands or open cursors outside of a transaction. Transactions have a well-defined lifetime, so attempting to use a transaction after it has completed throws exceptions. Also, transactions auto-commit and cannot be committed manually.</p>

  <p>This transaction model is really useful when you consider what might happen if a user opened two instances of your web app in two different tabs simultaneously. Without transactional operations, the two instances could interfere with each other's modifications. If you are not familiar with transactions in a database, read the <a class="link-https" href="https://en.wikipedia.org/wiki/Database_transaction">Wikipedia article on transactions</a>. Also see <a href="#transaction">transaction</a> under the Definitions section.</p>
 </li>
 <li>
  <p><strong>The IndexedDB API is mostly asynchronous.</strong> The API doesn't give you data by returning values; instead, you have to pass a callback function. You don't "store" a value into the database, or "retrieve" a value out of the database through synchronous means. Instead, you "request" that a database operation happens. You get notified by a DOM event when the operation finishes, and the type of event you get lets you know if the operation succeeded or failed. This sounds a little complicated at first, but there are sanity measures baked in. It's not that different from the way that <a href="/en-US/docs/Web/API/XMLHttpRequest">XMLHttpRequest</a> works.</p>
 </li>
 <li>
  <p><strong>IndexedDB uses a lot of requests. </strong>Requests are objects that receive the success or failure DOM events that were mentioned previously. They have <code>onsuccess</code> and <code>onerror</code> properties, and you can call <code>addEventListener()</code> and <code>removeEventListener()</code> on them. They also have <code>readyState</code>, <code>result</code>, and <code>errorCode</code> properties that tell you the status of the request. The <code>result</code> property is particularly magical, as it can be many different things, depending on how the request was generated (for example, an <code>IDBCursor</code> instance, or the key for a value that you just inserted into the database).</p>
 </li>
 <li>
  <p><strong>IndexedDB uses DOM events to notify you when results are available.</strong> DOM events always have a <code>type</code> property (in IndexedDB, it is most commonly set to <code>"success"</code> or <code>"error"</code>). DOM events also have a <code>target</code> property that indicates where the event is headed. In most cases, the <code>target</code> of an event is the <code>IDBRequest</code> object that was generated as a result of doing some database operation. Success events don't bubble up and they can't be canceled. Error events, on the other hand, do bubble, and can be cancelled. This is quite important, as error events abort whatever transactions they're running in, unless they are cancelled.</p>
 </li>
 <li>
  <p><strong>IndexedDB is object-oriented.</strong> IndexedDB is not a relational database with tables representing collections of rows and columns. This important and fundamental difference affects the way you design and build your applications.</p>

  <p>In a traditional relational data store, you would have a table that stores a collection of rows of data and columns of named types of data. IndexedDB, on the other hand, requires you to create an object store for a type of data and persist JavaScript objects to that store. Each object store can have a collection of indexes that makes it efficient to query and iterate across. If you are not familiar with object-oriented database management systems, read the <a href="https://en.wikipedia.org/wiki/Object_database">Wikipedia article on object database</a>.</p>
 </li>
 <li>
  <p><strong>IndexedDB does not use Structured Query Language (<abbr>SQL</abbr>).</strong> It uses queries on an index that produces a cursor, which you use to iterate across the result set. If you are not familiar with NoSQL systems, read the <a href="https://en.wikipedia.org/wiki/NoSQL">Wikipedia article on NoSQL</a>.</p>
 </li>
 <li>
  <p><strong>IndexedDB adheres to a same-origin policy</strong>. An origin is the domain, application layer protocol, and port of a URL of the document where the script is being executed. Each origin has its own associated set of databases. Every database has a name that identifies it within an origin.</p>

  <p>The security boundary imposed on IndexedDB prevents applications from accessing data with a different origin. For example, while an app or a page in <a href="https://www.example.com/app/">http://www.example.com/app/</a> can retrieve data from <a href="https://www.example.com/dir/">http://www.example.com/dir/</a>, because they have the same origin, it cannot retrieve data from <a href="https://www.example.com:8080/dir/">http://www.example.com:8080/dir/</a> (different port) or <a class="link-https" href="https://www.example.com/dir/">https://www.example.com/dir/</a> (different protocol), because they have different origins.</p>

  <div class="note"><p><strong>Note:</strong> Third party window content (e.g. {{htmlelement("iframe")}} content) can access the IndexedDB store for the origin it is embedded into, unless the browser is set to <a href="https://support.mozilla.org/en-US/kb/disable-third-party-cookies">never accept third party cookies</a> (see {{bug("1147821")}}.)</p></div>
 </li>
</ul>

<h3 id="limitations">Limitations</h3>

<p>IndexedDB is designed to cover most cases that need client-side storage. However, it is not designed for a few cases like the following:</p>

<ul>
 <li>Internationalized sorting. Not all languages sort strings in the same way, so internationalized sorting is not supported. While the database can't store data in a specific internationalized order, you can sort the data that you've read out of the database yourself. Note, however, that <a href="/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB#locale-aware_sorting">locale-aware sorting</a> has been allowed with an experimental flag enabled (currently for Firefox only) since Firefox 43.</li>
 <li>Synchronizing. The API is not designed to take care of synchronizing with a server-side database. You have to write code that synchronizes a client-side indexedDB database with a server-side database.</li>
 <li>Full text searching. The API does not have an equivalent of the <code>LIKE</code> operator in SQL.</li>
</ul>

<p>In addition, be aware that browsers can wipe out the database, such as in the following conditions:</p>

<ul>
 <li>The user requests a wipe out. Many browsers have settings that let users wipe all data stored for a given website, including cookies, bookmarks, stored passwords, and IndexedDB data.</li>
 <li>The browser is in private browsing mode. Some browsers, have "private browsing" (Firefox) or "incognito" (Chrome) modes. At the end of the session, the browser wipes out the database.</li>
 <li>The disk or quota limit has been reached.</li>
 <li>The data is corrupt.</li>
 <li>An incompatible change is made to the feature.</li>
</ul>

<p>The exact circumstances and browser capabilities change over time, but the general philosophy of the browser vendors is to make the best effort to keep the data when possible.</p>

<h2 id="core_terminology">Core terminology</h2>

<p>This section defines and explains core terminology relevant to understanding the IndexedDB API.</p>

<h3 id="database">Database</h3>

<h4>database</h4>

<p>A repository of information, typically comprising one or more <a href="#object_store"><em>object stores</em></a>. Each database must have the following:
 <ul>
  <li>Name. This identifies the database within a specific origin and stays constant throughout its lifetime. The name can be any string value (including an empty string).</li>
  <li>
   <p>Current <a href="#version"><em>version</em></a>. When a database is first created, its version is the integer 1 if not specified otherwise. Each database can have only one version at any given time.</p>
  </li>
 </ul>
</p>

<h4>database connection</h4>

<p>An operation created by opening a <em><a href="#database">database</a></em>. A given database can have multiple connections at the same time.</p>

<h4>durable</h4>

<p>In Firefox, IndexedDB used to be <strong>durable</strong>, meaning that in a readwrite transaction {{domxref("IDBTransaction.oncomplete")}} was fired only when all data was guaranteed to have been flushed to disk.</p>

<p>As of Firefox 40, IndexedDB transactions have relaxed durability guarantees to increase performance (see {{Bug("1112702")}}), which is the same behavior as other IndexedDB-supporting browsers. In this case the {{Event("complete")}} event is fired after the OS has been told to write the data but potentially before that data has actually been flushed to disk. The event may thus be delivered quicker than before, however, there exists a small chance that the entire transaction will be lost if the OS crashes or there is a loss of system power before the data is flushed to disk. Since such catastrophic events are rare, most consumers should not need to concern themselves further.</p>

<div class="note">
 <p><strong>Note:</strong> In Firefox, if you wish to ensure durability for some reason (e.g. you're storing critical data that cannot be recomputed later) you can force a transaction to flush to disk before delivering the <code>complete</code> event by creating a transaction using the experimental (non-standard) <code>readwriteflush</code> mode (see {{domxref("IDBDatabase.transaction")}}.) This is currently experimental, and can only be used if the <code>dom.indexedDB.experimental</code> pref is set to <code>true</code> in <code>about:config</code>.</p>
</div>

<h4>index</h4>

<p>An index is a specialized object store for looking up records in another object store, called the <em>referenced object store</em>. The index is a persistent key-value storage where the value part of its records is the key part of a record in the referenced object store. The records in an index are automatically populated whenever records in the referenced object store are inserted, updated, or deleted. Each record in an index can point to only one record in its referenced object store, but several indexes can reference the same object store. When the object store changes, all indexes that refer to the object store are automatically updated.</p>

<p>Alternatively, you can also look up records in an object store using the <a href="#key"> key</a><em>.</em></p>

<p>To learn more on using indexes, see <a href="/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB#using_an_index">Using IndexedDB</a>. For the reference documentation on index, see <a href="/en-US/docs/Web/API/IDBKeyRange" rel="internal">IDBKeyRange</a>.</p>

<h4>object store</h4>

<p>The mechanism by which data is stored in the database. The object store persistently holds records, which are key-value pairs. Records within an object store are sorted according to the <em><a href="#key">keys</a></em> in an ascending order.</p>

<p>Every object store must have a name that is unique within its database. The object store can optionally have a <em><a href="#key_generator">key generator</a></em> and a <em><a href="#key_path">key path</a></em>. If the object store has a key path, it is using <em><a href="#in-line_key">in-line keys</a></em>; otherwise, it is using <em><a href="#out-of-line_key">out-of-line keys</a></em>.</p>

<p>For the reference documentation on object store, see {{domxref("IDBObjectStore")}}.</p>

<h4>request</h4>

<p>The operation by which reading and writing on a database is done. Every request represents one read or write operation.</p>

<h4>transaction</h4>

<p>An atomic set of data-access and data-modification operations on a particular database. It is how you interact with the data in a database. In fact, any reading or changing of data in the database must happen in a transaction.</p>

<p>A database connection can have several active transactions associated with it at a time, so long as the writing transactions do not have overlapping <a href="#scope"><em>scopes</em></a>. The scope of transactions, which is defined at creation, determines which object stores the transaction can interact with and remains constant for the lifetime of the transaction. So, for example, if a database connection already has a writing transaction with a scope that just covers the <code>flyingMonkey</code> object store, you can start a second transaction with a scope of the <code>unicornCentaur</code> and <code>unicornPegasus</code> object stores. As for reading transactions, you can have several of them — even overlapping ones.</p>

<p>Transactions are expected to be short-lived, so the browser can terminate a transaction that takes too long, in order to free up storage resources that the long-running transaction has locked. You can abort the transaction, which rolls back the changes made to the database in the transaction. And you don't even have to wait for the transaction to start or be active to abort it.</p>

<p>The three modes of transactions are: <code>readwrite</code>, <code>readonly</code>, and <code>versionchange</code>. The only way to create and delete object stores and indexes is by using a <a href="/en-US/docs/Web/API/IDBDatabase/versionchange_event"><code>versionchange</code></a> transaction. To learn more about transaction types, see the reference article for <a href="/en-US/docs/Web/API/IndexedDB_API">IndexedDB</a>.</p>

<p>Because everything happens within a transaction, it is a very important concept in IndexedDB. To learn more about transactions, especially on how they relate to versioning, see {{domxref("IDBTransaction")}}, which also has reference documentation.</p>

<h4>version</h4>

<p>When a database is first created, its version is the integer 1. Each database has one version at a time; a database can't exist in multiple versions at once. The only way to change the version is by opening it with a greater version than the current one.</p>

<h3 id="key">Key and value</h3>

<h4>in-line key</h4>

<p>A key that is stored as part of the stored value. It is found using a <em>key path</em>. An in-line key can be generated using a generator. After the key has been generated, it can then be stored in the value using the key path or it can also be used as a key.</p>

<h4>key</h4>

<p>A data value by which stored values are organized and retrieved in the object store. The object store can derive the key from one of three sources: a <em><a href="#key_generator">key generator</a></em>, a <em><a href="#key_path">key path</a></em>, or an explicitly specified value. The key must be of a data type that has a number that is greater than the one before it. Each record in an object store must have a key that is unique within the same store, so you cannot have multiple records with the same key in a given object store.</p>

<p>A key can be one of the following types: <a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/String">string</a>, <a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date">date</a>, float, a binary blob, and <a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array">array</a>. For arrays, the key can range from an empty value to infinity. And you can include an array within an array.</p>

<p>Alternatively, you can also look up records in an object store using the <em><a href="#index">index</a>.</em></p>

<h4>key generator</h4>

<p>A mechanism for producing new keys in an ordered sequence. If an object store does not have a key generator, then the application must provide keys for records being stored. Generators are not shared between stores. This is more a browser implementation detail, because in web development, you don't really create or access key generators.</p>

<h4>key path</h4>

<p>Defines where the browser should extract the key from in the object store or index. A valid key path can include one of the following: an empty string, a JavaScript identifier, or multiple JavaScript identifiers separated by periods or an array containing any of those. It cannot include spaces.</p>

<h4>out-of-line key</h4>

<p>A key that is stored separately from the value being stored.</p>

<h4>value</h4>

<p>Each record has a value, which could include anything that can be expressed in JavaScript, including <a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean" rel="internal">boolean</a>, <a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number" rel="internal">number</a>, <a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/String">string</a>, <a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date">date</a>, <a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object">object</a>, <a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array" rel="internal">array</a>, <a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp" rel="internal">regexp</a>, <a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/undefined">undefined</a>, and null.</p>

<p>When an object or array is stored, the properties and values in that object or array can also be anything that is a valid value.</p>

<p><a href="/en-US/docs/Web/API/Blob">Blobs</a> and files can be stored, cf. <a href="https://dvcs.w3.org/hg/IndexedDB/raw-file/tip/Overview.html">specification</a>.</p>

<h3 id="range">Range and scope</h3>

<h4>cursor</h4>

<p>A mechanism for iterating over multiple records with a <em>key range</em>. The cursor has a source that indicates which index or object store it is iterating. It has a position within the range, and moves in a direction that is increasing or decreasing in the order of record keys. For the reference documentation on cursors, see {{domxref("IDBCursor")}}.</p>

<h4>key range</h4>

<p>A continuous interval over some data type used for keys. Records can be retrieved from object stores and indexes using keys or a range of keys. You can limit or filter the range using lower and upper bounds. For example, you can iterate over all values of a key between x and y.</p>

<p>For the reference documentation on key range, see {{domxref("IDBKeyRange")}}.</p>

<h4>scope</h4>

<p>The set of object stores and indexes to which a transaction applies. The scopes of read-only transactions can overlap and execute at the same time. On the other hand, the scopes of writing transactions cannot overlap. You can still start several transactions with the same scope at the same time, but they just queue up and execute one after another.</p>

<h2 id="next">Next steps</h2>

<p>With an understanding of IndexedDB’s key characteristics and core terminology under our belts, we can get to more concrete stuff. For a tutorial on how to use the API, see <a href="/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB">Using IndexedDB</a>.</p>

<h2 id="See_also">See also</h2>

<ul>
 <li><a href="https://www.w3.org/TR/IndexedDB/">Indexed Database API Specification</a></li>
 <li><a href="/en-US/docs/Web/API/IndexedDB_API">IndexedDB API Reference</a></li>
 <li><a href="/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB">Using IndexedDB</a></li>
 <li><a href="https://msdn.microsoft.com/en-us/scriptjunkie/gg679063.aspx">IndexedDB — The Store in Your Browser</a></li>
</ul>