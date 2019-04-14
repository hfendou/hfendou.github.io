

> [Hfendou.Blog](http://hfendou.gitbub.io) borned on 2018-08-02!<br><br>
> 日后将会持续地把学习历程及重要的信息记录这里！


# lucene

## lucene七大对象
```
Lucene core有七个包：analysis，document．index，queryParser，search，store，util。
1 analysis
Analysis包含一些内建的分析器，例如按空白字符分词的WhitespaceAnalyzer，添加了stopwrod过滤的StopAnalyzer，最常用的StandardAnalyzer。
2 document
Document．含文档的数据结构，例如document．定义了存储文档的数据结构，Field类定义了document．一个域。
3 index
Index包含了索引的读写类，例如对索引文件的segment进行写、合并、优化的IndexWriter类和对索引进行读取和删除操作的 IndexReader类，这里要注意的是不要被IndexReader这个名字误导，以为它是索引文件的读取类，实际上删除索引也是由它完成， IndexWriter只关心如何将索引写入一个个segment，并将它们合并优化；IndexReader则关注索引文件中各个文档的组织形式。
4 queryParser
QueryParser包含了解析查询语句的类，lucene的查询语句和sql语句有点类似，有各种保留字，按照一定的语法可以组成各种查询。 Lucene有很多种Query类，它们都继承自Query，执行各种特殊的查询，QueryParser的作用就是解析查询语句，按顺序调用各种 Query类查找出结果。
5 search
Search包含了从索引中搜索结果的各种类，例如刚才说的各种Query类，包括TermQuery、BooleanQuery等就在这个包里。
6 store
Store包含了索引的存储类，例如Directory定义了索引文件的存储结构，FSDirectory为存储在文件中的索引，RAMDirectory为存储在内存中的索引，MmapDirectory为使用内存映射的索引。
7 util
Util包含一些公共工具类，例如时间和字符串之间的转换工具。
```

### QueryParser
```
看了这么多Query，你可能会问：“不会让我自己组合各种Query吧，太麻烦了！”当然不会，lucene提供了一种类似于SQL语句的查询语句，我们姑且叫它lucene语句，通过它，你可以把各种查询一句话搞定，lucene会自动把它们查分成小块交给相应Query执行。下面我们对应每种 Query演示一下：
TermQuery可以用“field:key”方式，例如“content:lucene”。
BooleanQuery中‘与’用‘+’，‘或’用‘ ’，例如“content:java content：perl”。
WildcardQuery仍然用‘?’和‘*’，例如“content:use*”。
PhraseQuery用‘~’，例如“content:"中日"~5”。
PrefixQuery用‘*’，例如“中*”。
FuzzyQuery用‘~’，例如“content: wuzza ~”。
RangeQuery用‘[]’或‘{}’，前者表示闭区间，后者表示开区间，例如“time:[20060101 TO 20060130]”，注意TO区分大小写。
```

## lucene索引增删改查
``` c#
    public class LuceneBulid : ILuceneBulid
    {
        #region Identity
        private Logger logger = new Logger(typeof(LuceneBulid));
        #endregion Identity

        #region 批量BuildIndex 索引合并
        /// <summary>
        /// 批量创建索引(要求是统一的sourceflag，即目录是一致的)
        /// </summary>
        /// <param name="ciList">sourceflag统一的</param>
        /// <param name="pathSuffix">索引目录后缀，加在电商的路径后面，为空则为根目录.如sa\1</param>
        /// <param name="isCreate">默认为false 增量索引  true的时候删除原有索引</param>
        public void BuildIndex(List<Commodity> ciList, string pathSuffix = "", bool isCreate = false)
        {
            IndexWriter writer = null;
            try
            {
                if (ciList == null || ciList.Count == 0)
                {
                    return;
                }

                string rootIndexPath = StaticConstant.IndexPath;
                string indexPath = string.IsNullOrWhiteSpace(pathSuffix) ? rootIndexPath : string.Format("{0}\\{1}", rootIndexPath, pathSuffix);

                DirectoryInfo dirInfo = Directory.CreateDirectory(indexPath);
                LuceneIO.Directory directory = LuceneIO.FSDirectory.Open(dirInfo);
                writer = new IndexWriter(directory, new PanGuAnalyzer(), isCreate, IndexWriter.MaxFieldLength.LIMITED);
                //writer = new IndexWriter(directory, CreateAnalyzerWrapper(), isCreate, IndexWriter.MaxFieldLength.LIMITED);
                writer.SetMaxBufferedDocs(100);//控制写入一个新的segent前内存中保存的doc的数量 默认10  
                writer.MergeFactor = 100;//控制多个segment合并的频率，默认10
                writer.UseCompoundFile = true;//创建符合文件 减少索引文件数量

                ciList.ForEach(c => CreateCIIndex(writer, c));
            }
            finally
            {
                if (writer != null)
                {
                    //writer.Optimize(); 创建索引的时候不做合并  merge的时候处理
                    writer.Close();
                }
            }
        }

        /// <summary>
        /// 将索引合并到上级目录
        /// </summary>
        /// <param name="sourceDir">子文件夹名</param>
        public void MergeIndex(string[] childDirs)
        {
            Console.WriteLine("MergeIndex Start");
            IndexWriter writer = null;
            try
            {
                if (childDirs == null || childDirs.Length == 0) return;
                Analyzer analyzer = new StandardAnalyzer(Lucene.Net.Util.Version.LUCENE_30);
                string rootPath = StaticConstant.IndexPath;
                DirectoryInfo dirInfo = Directory.CreateDirectory(rootPath);
                LuceneIO.Directory directory = LuceneIO.FSDirectory.Open(dirInfo);
                writer = new IndexWriter(directory, analyzer, true, IndexWriter.MaxFieldLength.LIMITED);//删除原有的
                LuceneIO.Directory[] dirNo = childDirs.Select(dir => LuceneIO.FSDirectory.Open(Directory.CreateDirectory(string.Format("{0}\\{1}", rootPath, dir)))).ToArray();
                writer.MergeFactor = 100;//控制多个segment合并的频率，默认10
                writer.UseCompoundFile = true;//创建符合文件 减少索引文件数量
                writer.AddIndexesNoOptimize(dirNo);
            }
            finally
            {
                if (writer != null)
                {
                    writer.Optimize();
                    writer.Close();
                }
                Console.WriteLine("MergeIndex End");
            }
        }

        //Field.Store.YES:存储字段值（未分词前的字段值）        
        //Field.Store.NO:不存储,存储与索引没有关系         
        //Field.Store.COMPRESS:压缩存储,用于长文本或二进制，但性能受损         
        //Field.Index.ANALYZED:分词建索引         
        //Field.Index.ANALYZED_NO_NORMS:分词建索引，但是Field的值不像通常那样被保存，而是只取一个byte，这样节约存储空间         
        //Field.Index.NOT_ANALYZED:不分词且索引         
        //Field.Index.NOT_ANALYZED_NO_NORMS:不分词建索引，Field的值去一个byte保存         
        //TermVector表示文档的条目（由一个Document和Field定位）和它们在当前文档中所出现的次数         
        //Field.TermVector.YES:为每个文档（Document）存储该字段的TermVector         
        //Field.TermVector.NO:不存储TermVector         
        // Field.TermVector.WITH_POSITIONS:存储位置        
        //Field.TermVector.WITH_OFFSETS:存储偏移量         
        //Field.TermVector.WITH_POSITIONS_OFFSETS:存储位置和偏移量
        #endregion 批量BuildIndex 索引合并

        #region 单个/批量索引增删改
        /// <summary>
        /// 新增一条数据的索引
        /// </summary>
        /// <param name="ci"></param>
        public void InsertIndex(Commodity ci)
        {
            IndexWriter writer = null;
            try
            {
                if (ci == null) return;
                string rootIndexPath = StaticConstant.IndexPath;
                DirectoryInfo dirInfo = Directory.CreateDirectory(rootIndexPath);

                bool isCreate = dirInfo.GetFiles().Count() == 0;//下面没有文件则为新建索引 
                LuceneIO.Directory directory = LuceneIO.FSDirectory.Open(dirInfo);
                writer = new IndexWriter(directory, CreateAnalyzerWrapper(), isCreate, IndexWriter.MaxFieldLength.LIMITED);
                writer.MergeFactor = 100;//控制多个segment合并的频率，默认10
                writer.UseCompoundFile = true;//创建符合文件 减少索引文件数量
                CreateCIIndex(writer, ci);
            }
            catch (Exception ex)
            {
                logger.Error("InsertIndex异常", ex);
                throw ex;
            }
            finally
            {
                if (writer != null)
                {
                    //if (fileNum > 50)
                    //    writer.Optimize();
                    writer.Close();
                }
            }
        }

        /// <summary>
        /// 批量新增数据的索引
        /// </summary>
        /// <param name="ciList"></param>
        public void InsertIndexMuti(List<Commodity> ciList)
        {
            BuildIndex(ciList, "", false);
        }

        /// <summary>
        /// 批量删除数据的索引
        /// </summary>
        /// <param name="ciList"></param>
        public void DeleteIndexMuti(List<Commodity> ciList)
        {
            IndexReader reader = null;
            try
            {
                if (ciList == null || ciList.Count == 0) return;
                Analyzer analyzer = new StandardAnalyzer(Lucene.Net.Util.Version.LUCENE_30);
                string rootIndexPath = StaticConstant.IndexPath;
                DirectoryInfo dirInfo = Directory.CreateDirectory(rootIndexPath);
                LuceneIO.Directory directory = LuceneIO.FSDirectory.Open(dirInfo);
                reader = IndexReader.Open(directory, false);
                foreach (Commodity ci in ciList)
                {
                    reader.DeleteDocuments(new Term("productid", ci.ProductId.ToString()));
                }
            }
            catch (Exception ex)
            {
                logger.Error("DeleteIndex异常", ex);
                throw ex;
            }
            finally
            {
                if (reader != null)
                {
                    reader.Dispose();
                }
            }
        }

        /// <summary>
        /// 删除多条数据的索引
        /// </summary>
        /// <param name="ci"></param>
        public void DeleteIndex(Commodity ci)
        {
            IndexReader reader = null;
            try
            {
                if (ci == null) return;
                Analyzer analyzer = new StandardAnalyzer(Lucene.Net.Util.Version.LUCENE_30);
                string rootIndexPath = StaticConstant.IndexPath;
                DirectoryInfo dirInfo = Directory.CreateDirectory(rootIndexPath);
                LuceneIO.Directory directory = LuceneIO.FSDirectory.Open(dirInfo);
                reader = IndexReader.Open(directory, false);
                reader.DeleteDocuments(new Term("productid", ci.ProductId.ToString()));
            }
            catch (Exception ex)
            {

                logger.Error("DeleteIndex异常", ex);
                throw ex;
            }
            finally
            {
                if (reader != null)
                {
                    reader.Dispose();
                }
            }
        }

        ///// <summary>
        ///// 更新一条数据的索引
        ///// </summary>
        //public void UpdateIndex(Commodity ci)
        //{
        //    DeleteIndex(ci);
        //    InsertIndex(ci);
        //}

        /// <summary>
        /// 更新一条数据的索引
        /// </summary>
        /// <param name="ci"></param>
        public void UpdateIndex(Commodity ci)
        {
            IndexWriter writer = null;
            try
            {
                if (ci == null) return;
                string rootIndexPath = StaticConstant.IndexPath;
                DirectoryInfo dirInfo = Directory.CreateDirectory(rootIndexPath);

                bool isCreate = dirInfo.GetFiles().Count() == 0;//下面没有文件则为新建索引 
                LuceneIO.Directory directory = LuceneIO.FSDirectory.Open(dirInfo);
                writer = new IndexWriter(directory, CreateAnalyzerWrapper(), isCreate, IndexWriter.MaxFieldLength.LIMITED);
                writer.MergeFactor = 100;//控制多个segment合并的频率，默认10
                writer.UseCompoundFile = true;//创建符合文件 减少索引文件数量
                writer.UpdateDocument(new Term("productid", ci.ProductId.ToString()), ParseCItoDoc(ci));
            }
            catch (Exception ex)
            {
                logger.Error("InsertIndex异常", ex);
                throw ex;
            }
            finally
            {
                if (writer != null)
                {
                    //if (fileNum > 50)
                    //    writer.Optimize();
                    writer.Close();
                }
            }
        }

        /// <summary>
        /// 批量更新数据的索引
        /// </summary>
        /// <param name="ciList">sourceflag统一的</param>
        public void UpdateIndexMuti(List<Commodity> ciList)
        {
            IndexWriter writer = null;
            try
            {
                if (ciList == null || ciList.Count == 0) return;
                string rootIndexPath = StaticConstant.IndexPath;
                DirectoryInfo dirInfo = Directory.CreateDirectory(rootIndexPath);

                bool isCreate = dirInfo.GetFiles().Count() == 0;//下面没有文件则为新建索引 
                LuceneIO.Directory directory = LuceneIO.FSDirectory.Open(dirInfo);
                writer = new IndexWriter(directory, CreateAnalyzerWrapper(), isCreate, IndexWriter.MaxFieldLength.LIMITED);
                writer.MergeFactor = 50;//控制多个segment合并的频率，默认10
                writer.UseCompoundFile = true;//创建符合文件 减少索引文件数量
                foreach (Commodity ci in ciList)
                {
                    writer.UpdateDocument(new Term("productid", ci.ProductId.ToString()), ParseCItoDoc(ci));
                }
            }
            catch (Exception ex)
            {
                logger.Error("InsertIndex异常", ex);
                throw ex;
            }
            finally
            {
                if (writer != null)
                {
                    //if (fileNum > 50)
                    //    writer.Optimize();
                    writer.Close();
                }
            }
        }
        #endregion 单个索引增删改

        #region PrivateMethod
        /// <summary>
        /// 创建分析器
        /// </summary>
        /// <returns></returns>
        private PerFieldAnalyzerWrapper CreateAnalyzerWrapper()
        {
            Analyzer analyzer = new StandardAnalyzer(Lucene.Net.Util.Version.LUCENE_30);

            PerFieldAnalyzerWrapper analyzerWrapper = new PerFieldAnalyzerWrapper(analyzer);
            analyzerWrapper.AddAnalyzer("title", new PanGuAnalyzer());
            analyzerWrapper.AddAnalyzer("categoryid", new StandardAnalyzer(Lucene.Net.Util.Version.LUCENE_30));
            return analyzerWrapper;
        }

        /// <summary>
        /// 创建索引
        /// </summary>
        /// <param name="analyzer"></param>
        /// <param name="title"></param>
        /// <param name="content"></param>
        private void CreateCIIndex(IndexWriter writer, Commodity ci)
        {
            try
            {
                writer.AddDocument(ParseCItoDoc(ci));
            }
            catch (Exception ex)
            {
                logger.Error("CreateCIIndex异常", ex);
                throw ex;
            }
        }

        /// <summary>
        /// 将Commodity转换成doc
        /// </summary>
        /// <param name="ci"></param>
        /// <returns></returns>
        private Document ParseCItoDoc(Commodity ci)
        {
            Document doc = new Document();

            doc.Add(new Field("id", ci.Id.ToString(), Field.Store.YES, Field.Index.NOT_ANALYZED));
            doc.Add(new Field("title", ci.Title, Field.Store.YES, Field.Index.ANALYZED));//盘古分词
            doc.Add(new Field("productid", ci.ProductId.ToString(), Field.Store.YES, Field.Index.NOT_ANALYZED));
            doc.Add(new Field("categoryid", ci.CategoryId.ToString(), Field.Store.YES, Field.Index.NOT_ANALYZED));
            doc.Add(new Field("imageurl", ci.ImageUrl, Field.Store.YES, Field.Index.NOT_ANALYZED));
            doc.Add(new Field("url", ci.Url, Field.Store.YES, Field.Index.NOT_ANALYZED));
            doc.Add(new NumericField("price", Field.Store.YES, true).SetFloatValue((float)ci.Price));
            return doc;
        }

        #endregion PrivateMethod
    }

```


## 搜索实例

``` c#
			string indexPath = System.Configuration.ConfigurationManager.AppSettings["LuceneNetDir"];
			string searchString = Request["txtSearch"];
			List<string> list = LuceneCommon.PanGuSplitWord("Title",searchString);//对用户输入的搜索条件进行拆分。
			FSDirectory directory = FSDirectory.Open(new DirectoryInfo(indexPath), new NoLockFactory());
			IndexReader reader = IndexReader.Open(directory, true);
			IndexSearcher searcher = new IndexSearcher(reader);
			//搜索条件
			PhraseQuery queryBody = new PhraseQuery();
			////多部分查询
			//PhraseQuery queryTitle = new PhraseQuery();
			//先用空格，让用户去分词，空格分隔的就是词“计算机   专业”
			foreach (string word in list)
			{
				queryBody.Add(new Term("Price", word));
				//queryTitle.Add(new Term("Title", word));
			}
			//多个查询条件的词之间的最大距离.在文章中相隔太远 也就无意义.（例如 “大学生”这个查询条件和"简历"这个查询条件之间如果间隔的词太多也就没有意义了。）
			queryBody.Slop = 100;
			//queryTitle.SetSlop(100);

			#region 多部分查询

			//BooleanQuery query = new BooleanQuery();
			//query.Add(queryTitle, BooleanClause.Occur.SHOULD);
			//query.Add(queryBody, BooleanClause.Occur.SHOULD);

			#endregion

			//TopScoreDocCollector是盛放查询结果的容器
			TopScoreDocCollector collector = TopScoreDocCollector.Create(1000, true);
			//根据query查询条件进行查询，查询结果放入collector容器
			searcher.Search(queryBody, null, collector);
			//得到所有查询结果中的文档,GetTotalHits():表示总条数   TopDocs(300, 20);//表示得到300（从300开始），到320（结束）的文档内容.
			ScoreDoc[] docs = collector.TopDocs(0, collector.TotalHits).ScoreDocs;
			//可以用来实现分页功能
			List<ContentViewModel> viewModelList = new List<ContentViewModel>();
			for (int i = 0; i < docs.Length; i++)
			{
				//搜索ScoreDoc[]只能获得文档的id,这样不会把查询结果的Document一次性加载到内存中。降低了内存压力，需要获得文档的详细内容的时候通过searcher.Doc来根据文档id来获得文档的详细内容对象Document.
				ContentViewModel viewModel = new ContentViewModel();
				int docId = docs[i].Doc;//得到查询结果文档的id（Lucene内部分配的id）
				Document doc = searcher.Doc(docId);//找到文档id对应的文档详细信息
				viewModel.Id = Convert.ToInt32(doc.Get("Id"));// 取出放进字段的值
				viewModel.Title = doc.Get("Title");
				viewModel.Content = LuceneCommon.CreateHightLight(searchString, doc.Get("Price"));//将搜索的关键字高亮显示。
				viewModelList.Add(viewModel);
			}
			//先将搜索的词插入到明细表。
			SearchDetails searchDetail = new SearchDetails();
			searchDetail.Id = Guid.NewGuid();
			searchDetail.KeyWords = Request["txtSearch"];
			searchDetail.SearchTime = DateTime.Now;
			SearchDetailsService.AddEntity(searchDetail);

			return viewModelList;

```

