```
package com.johnny.lucene01.index;

import java.io.File;
import java.io.FileReader;
import java.io.IOException;

import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.StringField;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.queryparser.classic.QueryParser;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.TopDocs;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;
import org.apache.lucene.util.Version;

/**

- @author Johnny
- @description: 
- 依赖jar:Lucene-core，Lucene-analysis（使用标准分词器做测试），Lucene-queryParser
- 作用：简单的索引建立和读取步骤
*/
public class HelloLucene {
public static Version luceneVersion = Version.LUCENE_4_10_2;
/**
  - 建立索引
*/
public void createIndex(){
IndexWriter writer = null;
try{
    //1、创建Directory
    //Directory directory = new RAMDirectory();//创建内存directory
    Directory directory = FSDirectory.open(new File("/Users/ChinaMWorld/Desktop/WorkSpace/Johnny/lucene/lucene/src/main/java/resources/lucene01/index/"));//在硬盘上生成Directory
    //2、创建IndexWriter
    IndexWriterConfig iwConfig = new IndexWriterConfig(luceneVersion, new StandardAnalyzer());
    writer = new IndexWriter(directory, iwConfig);
    //3、创建document对象
    Document document = null;
    //4、为document添加field对象
    File f = new File("/Users/ChinaMWorld/Desktop/WorkSpace/Johnny/lucene/lucene/src/main/java/resources/lucene01/file/");//索引源文件位置
    for(File file:f.listFiles()){
        document = new Document();
        document.add(new StringField("fileName", file.getName(),Field.Store.YES));//store是否存储
        document.add(new StringField("path", file.getPath(),Field.Store.YES));
        document.add(new TextField("content", new FileReader(file)));//textField内容会进行分词
        //5、通过IndexWriter添加文档到索引中
        writer.addDocument(document);
    }
}catch(Exception e){
    e.printStackTrace();
}finally{
    //6、使用完成后需要将writer进行关闭
    try {
        writer.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
}
/**
  - 查询内容
*/
public void indexSearch(){
DirectoryReader reader = null;
try{
//            1、创建Directory
     Directory directory = FSDirectory.open(new File("/Users/ChinaMWorld/Desktop/WorkSpace/Johnny/lucene/lucene/src/main/java/resources/lucene01/index/"));//在硬盘上生成Directory
//            2、创建IndexReader
     reader = DirectoryReader.open(directory);
//            3、根据IndexWriter创建IndexSearcher
     IndexSearcher searcher =  new IndexSearcher(reader);
//            4、创建搜索的query
//            创建parse用来确定搜索的内容，第二个参数表示搜索的域
     QueryParser parser = new QueryParser("content",new StandardAnalyzer());//content表示搜索的域或者说字段
     Query query = parser.parse("java");//java表示被搜索的内容
//            5、根据Searcher返回TopDocs
     TopDocs tds = searcher.search(query, 10);//查询10条记录
//            6、根据TopDocs获取ScoreDoc
     ScoreDoc[] sds = tds.scoreDocs;
//            7、根据Searcher和ScoreDoc获取搜索到的document对象
     for(ScoreDoc sd:sds){
         Document d = searcher.doc(sd.doc);
//                    8、根据document对象获取查询的字段值
         /**  查询结果中content为空，是因为索引中没有存储content的内容，需要根据索引ID从原文件中获取content**/
         System.out.println(d.get("fileName")+",,,"+d.get("content")+",,,"+d.get("path"));
     }
    }catch(Exception e){
    e.printStackTrace();
}finally{
    //9、关闭reader
    try {
        reader.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
}

}



```

