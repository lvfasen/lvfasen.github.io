#ElasticSearch

##java API

```java
import java.io.IOException;
import java.util.HashMap;

import org.apache.http.HttpHost;
import org.elasticsearch.ElasticsearchException;
import org.elasticsearch.action.DocWriteResponse;
import org.elasticsearch.action.admin.indices.delete.DeleteIndexRequest;
import org.elasticsearch.action.admin.indices.settings.get.GetSettingsRequest;
import org.elasticsearch.action.admin.indices.settings.get.GetSettingsResponse;
import org.elasticsearch.action.admin.indices.settings.put.UpdateSettingsRequest;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.get.GetRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.client.indices.CreateIndexRequest;
import org.elasticsearch.client.indices.CreateIndexResponse;
import org.elasticsearch.client.indices.GetIndexRequest;
import org.elasticsearch.common.collect.ImmutableOpenMap;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.common.xcontent.XContentType;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.index.reindex.DeleteByQueryRequest;
import org.elasticsearch.index.reindex.UpdateByQueryRequest;
import org.elasticsearch.rest.RestStatus;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.elasticsearch.search.builder.SearchSourceBuilder;


public class Helaticsearch {
    private RestHighLevelClient client;

public Helaticsearch() {
	client = new RestHighLevelClient(
	          RestClient.builder(new HttpHost("localhost", 9200, "http")));
}
public void close() {
	if(client != null) {
		try {
			client.close();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}

// 创建索引 
public void createdIndex(String index) {
	
	CreateIndexRequest request = new CreateIndexRequest(index);
	request.settings(Settings.builder().put("index.number_of_shards", 3));
	
	try {
		client.indices().create(request, RequestOptions.DEFAULT);
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}finally {
		close();
	}
	
}

// 判断索引是否存在
public boolean existsIndex(String index) {
	boolean exists = false;
	GetIndexRequest getIndexRequest = new GetIndexRequest(index);
	try {
		exists =  client.indices().exists(getIndexRequest, RequestOptions.DEFAULT);
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}finally {
		close();
	}
	return exists;
}

// 修改索引
public void updateIndex(String oldIndex, String newIndex) {
	UpdateSettingsRequest updateSettingsRequest = new UpdateSettingsRequest(oldIndex);
	updateSettingsRequest.settings(Settings.builder().put(newIndex , true).build());
	
	try {
		client.indices().putSettings(updateSettingsRequest, RequestOptions.DEFAULT);
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}finally {
		close();
	}
}

// 删除索引
public void deleteIndex(String index) {
	DeleteIndexRequest requst = new DeleteIndexRequest(index);
	try {
		client.indices().delete(requst, RequestOptions.DEFAULT);
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}finally {
		close();
	}
}

// 查看索引
public void getIndex(String index) {
	GetSettingsRequest request = new GetSettingsRequest().indices(index);
	
	try {
		GetSettingsResponse response = client.indices().getSettings(request, RequestOptions.DEFAULT);
		ImmutableOpenMap<String,Settings> indexToSettings = response.getIndexToSettings();
		System.out.println(index);
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}finally {
		close();
	}
}

// 创建文档
public void createDoc(String index) {
	IndexRequest request = new IndexRequest(index);

	request.source("{\"name\": \"Tom\",\"age\": 12}", XContentType.JSON);
	
	try {
		IndexResponse response = client.index(request, RequestOptions.DEFAULT);
		String id = response.getId();
		String indexName = response.getIndex();
		
		if(response.getResult() == DocWriteResponse.Result.CREATED) {
			System.out.println("添加成功");
		}
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}finally {
		close();
	}	
}

// DeleteRequest删除文档
public void deleteDoc(String indexName, String documentId) {
	DeleteRequest requust= new DeleteRequest(indexName, documentId);
	try {
		client.delete(requust, RequestOptions.DEFAULT);
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}finally {
		close();
	}
}
// DeleteByQueryRequest删除文档
public void deleteByQueryDoc() {
	DeleteByQueryRequest request = new DeleteByQueryRequest();
}

// UpdateRequest更新文档
public void updateDoc(String indexName, String documentId) {
	UpdateRequest request = new UpdateRequest(indexName, documentId);
	String jsonString = "{" +
	        "\"user\":\"tom\"" +
	        "}";
	request.doc(jsonString, XContentType.JSON);
	try {
		client.update(request, RequestOptions.DEFAULT);
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}finally {
		close();
	}
}
// UpdateByQueryRequest更新文档
public void updateByQueryDoc() {
	UpdateByQueryRequest request= new UpdateByQueryRequest();
}

// 判断某个文档是否存在
public void existsDoc() {
	GetRequest request = new GetRequest();
	
	try {
		client.exists(request, RequestOptions.DEFAULT);
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}finally {
		close();
	}
}
// 获取索引下某个文档
public void getDoc(String indexName, String documentId) {
	GetRequest getRequest = new GetRequest(indexName, documentId);
	try {
		GetResponse response = client.get(getRequest, RequestOptions.DEFAULT);
		String sourceAsString = response.getSourceAsString();
		System.out.println(sourceAsString);
	} catch (ElasticsearchException e) {
		// TODO Auto-generated catch block
		if(e.status() == RestStatus.NOT_FOUND) {
			
		}
	}catch (IOException e) {
		// TODO: handle exception
		e.printStackTrace();
	}finally {
		close();
	}
	
}
// 批量操作文档
public void createDocs() {
	BulkRequest request = new BulkRequest("doc_elastic");
	String jsonString = "{" +
	        "\"user\":\"kimchy\"," +
	        "\"postDate\":\"2013-01-30\"," +
	        "\"message\":\"trying out Elasticsearch\"" +
	        "}";
	request.add(new IndexRequest().id("1").source(jsonString, XContentType.JSON));
	
	try {
		client.bulk(request, RequestOptions.DEFAULT);
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}finally {
		close();
	}
}

// matchAllQuery() 匹配指定索引下的所有文档
public void serchAll() {
	SearchRequest searchRequest = new SearchRequest("doc_elastic");
	
	SearchSourceBuilder builder = new SearchSourceBuilder();
	builder.query(QueryBuilders.matchAllQuery());
	
	searchRequest.source(builder);
	
	try {
		SearchResponse response = client.search(searchRequest, RequestOptions.DEFAULT);
		SearchHits hits = response.getHits();
		
		SearchHit[] hits2 = hits.getHits();
		for(SearchHit hit : hits2) {
			String sourceAsString = hit.getSourceAsString();

			System.out.println(sourceAsString);
		}
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}finally {
		close();
	}
	
}
public static void main(String[] args) {
     RestHighLevelClient client = null;
    try {
      // 链接es
      client = new RestHighLevelClient(
          RestClient.builder(new HttpHost("localhost", 9200, "http")));

      CreateIndexRequest request = new CreateIndexRequest("elastic_source2");
      request.settings(Settings.builder().put("index.number_of_shards", 3));
           HashMap<String, Object> message = new HashMap<>();
      message.put("type","text");
      HashMap<String, Object> name = new HashMap<>();
      name.put("type","text");
      HashMap<String, Object> age = new HashMap<>();
      age.put("type","integer");
       
      HashMap<String, Object> properties = new HashMap<>();
      properties.put("message",message);
      properties.put("name", name);
      properties.put("age", age);
      
      HashMap<String, Object> mapping = new HashMap<>();
      mapping.put("properties", properties);

      request.mapping(mapping);
        //	      HashMap<String, Object> source = new HashMap<>();
        //	      source.put("name", "zhangsan");
        //	      source.put("age", 24);
        //	      source.put("message", "test");
	  String jsonString = "{" +
	    	        "\"user\":\"kimchy\"," +
	    	        "\"postDate\":\"2013-01-30\"," +
	    	        "\"message\":\"trying out Elasticsearch\"" +
	    	        "}";
           request.source(jsonString, XContentType.JSON);
         CreateIndexResponse createIndexResponse = client.indices()
          .create(request, RequestOptions.DEFAULT);
      
      System.out.println("创建索引" + createIndexResponse.isAcknowledged());
    }catch (IOException e){
      e.printStackTrace();
    }finally {
      if(client != null)
		try {
			client.close();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
    }
  }
}     
```



