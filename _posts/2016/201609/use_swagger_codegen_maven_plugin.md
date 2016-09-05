# 使用swagger codegen maven plugin生成retrofit2代码
### 创建一个maven工程，在pom文件中引入plugin，如下

	<build>
		<plugins>
			<plugin>
			    <groupId>io.swagger</groupId>
			    <artifactId>swagger-codegen-maven-plugin</artifactId>
			    <version>2.2.1</version>
			    <executions>
			        <execution>
			            <goals>
			                <goal>generate</goal>
			            </goals>
			            <configuration>
			                <inputSpec>src/main/resources/swagger.yaml</inputSpec>
			                <language>java</language>
							<library>retrofit2</library>
			                <apiPackage>xyz.lament.api</apiPackage>
			                <modelPackage>xyz.lament.model</modelPackage>
			                <groupId>xyz.lament</groupId>
			                <artifactId>retrofitclient</artifactId>
			                <artifactVersion>0.0.1</artifactVersion>
			                <output>${basedir}/../retrofitClient</output>
			            </configuration>
			            <phase>generate-sources</phase>
			        </execution>
			    </executions>
			</plugin>
		</plugins>
	</build>
***
> 上面的配置中swagger.yaml是自己定义的restful接口描述文件。  
> language指定了生成的客户端代码的语言，library指定生成retrofit2形式的代码。  
> output指定生成代码的保存路径。  
> 执行Maven generate-sources，即可生成retrofit2 client的maven工程retrofitClient  
> maven install该工程。

### 在工程中使用restfulClient
> 新建一个maven工程，引入刚才install的restfulClient-0.01.jar

    <dependencies>  
      <dependency>
        <groupId>xyz.lament</groupId>
        <artifactId>retrofitclient</artifactId>      
        <version>0.0.1</version>
      </dependency>

      <dependency>
        <groupId>com.squareup.retrofit2</groupId>
        <artifactId>converter-jackson</artifactId>
        <version>2.1.0</version>
        </dependency>
    </dependencies>

> 在java代码中调用接口
  package lament.tryRetrofit;

    import java.io.IOException;

    import retrofit2.Call;
    import retrofit2.Response;
    import retrofit2.Retrofit;
    import retrofit2.converter.jackson.JacksonConverterFactory;
    import xyz.lament.ApiClient;
    import xyz.lament.api.GreetingApi;
    import xyz.lament.model.Greeting;

    public class App
    {
      public static void main( String[] args )
      {
        ApiClient apiClient = new ApiClient();
        Retrofit.Builder builder = new Retrofit.Builder()
            .baseUrl("http://localhost:8080/")
            .addConverterFactory(JacksonConverterFactory.create());
        apiClient.setAdapterBuilder(builder);

        GreetingApi service = apiClient.createService(GreetingApi.class);

        Call<Greeting> userCall = service.greeting("jimi");
        try
        {
          Response<Greeting> resultResponse = userCall.execute();
          System.out.println(resultResponse.body().toString());
        }
        catch (IOException e)
        {
        e.printStackTrace();
        }
      }
    }

> 执行，相当于调用`curl http://localhost:8080/greeting?name=jimi`  
> [gs-rest-service](https://git.oschina.net/LamentOfTheHighborne/sprintBootRestExample)工程提供restful接口。  
> [useswaggercodegen](https://git.oschina.net/LamentOfTheHighborne/useswaggercodegen)工程使用swagger-codegen-maven-plugin插件生成http调用的客户端代码。  
> [tryRetrofit](https://git.oschina.net/LamentOfTheHighborne/tryRetrofit)工程使用生成的retrofit2代码进行restful接口的调用。
