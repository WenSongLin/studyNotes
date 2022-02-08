# 0626工具:excel,word,fastdfs

## 1.excel的读写

在项目中做数据的导入导出.

导出与查询类似的效果,相比于查询把不做分页,查询结果写入excel文件,让用户下载.

导入就导入项目的基础数据(员工信息,商品产品,产品的信息),先上传excel,再进行excel的读取操作进行数据入库.(入库过程中,切记校验数据重复,切记把失败的记录返回给操作人)

excel操作的组件库:POI(原生的组件),ali-easyexcel,hutool-excelUtil

- 添加poi依赖

```
        <!--poi依赖库-->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>${poi.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-scratchpad</artifactId>
            <version>${poi.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-excelant</artifactId>
            <version>${poi.version}</version>
        </dependency>
```

- 进行读的实现

```
Workbook book = WorkbookFactory.create(InputStream in)
//TODO getSeetAt(int i)  getRow(int i)   getCell(int i)
```

- 进行写

```
Workbook book = new XSSFWorkbook()
//TODO createSheet,createRow,createCell
book.write(OutputStream out)
```



## 2.word生成

在项目中需要进行电子合同的生成,打印word文件.一般用在政府系统中.

基于freemarker模板引擎生成word.

- 创建word模板,通过${}表达式来指定动态数据.
- 添加freemarker依赖包.





freemarker语法:

获取动态数据:${}

循环标签:<#list list集合 as 循环变量></#list>





