
# SyncDirectory for Lucene.Net

works with Lucene.net 4.8 beta and targets .net standard 2.0

## Project description

Lucene.Net is a robust open source search technology which has an abstract interface called a Directory for defining how the index is stored. SyncDirectory is an implementation of that interface which allows you to use two FSDirectorys, one primary directory for permanent storage in the network based Azure file storage and one cache directory in the fast volatile storage.

## About
This project is meant to improve the experience of using Lucene.Net in Azure.

## SyncDirectory
SyncDirectory uses a local Directory on the fast volatile local storage of an App Service instance (storage not shared between instances when scaling) to cache files as they are created and automatically pushes them to another directory on the permanent, shared and much slower network based storage as appropriate. when new instances of your scaled Azure App Service startup, they will sync the master index on the slow but permanent network based storage to the fast volatile local index.

read more about the Azure App Service file system [here](https://github.com/projectkudu/kudu/wiki/Understanding-the-Azure-App-Service-file-system)

This Directory implementation is supposed to work with one app service instance adding documents to an index, and 1..N searcher instances (scaled app service) searching over the catalog.

To add documents to a catalog is as simple as

```cs
string primaryPath = ContentRootPath+ @"\App_Data\PrimaryIndex";
string cachePath = @"D:\local\Temp\CacheIndex";

var syncDirectory = new SyncDirectory(primaryPath, cachePath);

var indexWriterConfig = new IndexWriterConfig(
	Lucene.Net.Util.LuceneVersion.LUCENE_48,
	new StandardAnalyzer(Lucene.Net.Util.LuceneVersion.LUCENE_48));

var indexWriter = new IndexWriter(syncDirectory, indexWriterConfig);
            
var doc = new Document();

doc.Add(new Field("id", DateTime.Now.ToFileTimeUtc().ToString(), Field.Store.YES, Field.Index.TOKENIZED, Field.TermVector.NO));
doc.Add(new Field("Title", "this is my title", Field.Store.YES, Field.Index.TOKENIZED, Field.TermVector.NO));
doc.Add(new Field("Body", "This is my body", Field.Store.YES, Field.Index.TOKENIZED, Field.TermVector.NO));

indexWriter.AddDocument(doc);
indexWriter.Close();
```


And searching is as easy as:

```cs
var ireader = DirectoryReader.Open(syncDirectory);
var searcher = new IndexSearcher(ireader);
                    
var parser = new Lucene.Net.QueryParsers.Classic.QueryParser(Lucene.Net.Util.LuceneVersion.LUCENE_48, "Body", new StandardAnalyzer(Lucene.Net.Util.LuceneVersion.LUCENE_48));
var query = parser.Parse("Title:(Dog AND Cat)");

var hits = searcher.Search(query,100);
for (int i = 0; i < hits.Length(); i++)
{
    var doc = hits.Doc(i);
    Console.WriteLine(doc.GetField("Title").StringValue());
}
```            

## License

ML-PL
