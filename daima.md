你是编程导航平台（编程导航–做您编程路上的导航员)的文章创作者，现在你想要把自己在这个平台上发布的所有文章导出到自己的网站，并且根据文章的内容给每篇文章打上对应的多个标签(比如Java、Dubbo)，然后将这些文章按照标签分层级地进行归档，比如{"Java":["Java文章"，("Dubbo": ["Dubbo文章"]}，便于之后的阅读和搜索。

问题:原平台不提供导出、打标签、分层级归档、搜索等功能，请问如何自主设计实现上述系统。请尽量清晰详细地阐述思路，能具体实现更加分。



具体写了一部分代码，目前实现了爬取到Java、Dubbo关键字的效果。

> 效果展示
>
> s1 = ["Iterator","Java"] 这是因为我的贴子里没有Dubbo关键字

我的想法如下：

分析了文章接口地址 https://www.code-nav.cn/api/post/list/page/vo

可以针对该地址获取到用户发帖的所有数据data，而数据又主要分为title和content。

针对content里的内容，可以用正则匹配，如果是匹配到有类似关键字的开头，就可以加入到一个集合中，考虑到后面也可能会出现重复的关键字，这里使用TreeSet，但是同样也碰到问题，先进的关键字被拍到了后面，这里我的想法是可以使用栈结构(不知道会不会将问题复杂化)。

后面部分就还没有用代码实现，我的想法是：

既然已经拿到关键字数组，那就可以将他按结构转化，例如可以拼接括号，大括号这样的，实现到{"Java":["Java文章"，("Dubbo": ["Dubbo文章"]}的效果。

具体实现的代码如下：

```java
 @Test
    void contextLoads() throws IOException, JSONException {
        //1.获取数据
        String json = "{\"current\":1,\"pageSize\":8,\"sortField\":\"createTime\",\"sortOrder\":\"descend\",\"userId\":1610965721351716865,\"reviewStatus\":1}";
        String url = "https://www.code-nav.cn/api/post/list/page/vo";
        // https://www.code-nav.cn/api/post/list/page/vo
        String result2 = HttpRequest.post(url)
                .body(json)
                .execute().body();
        System.out.println(result2);
        //2.json 转对象
        Map<String, Object> map = JSONUtil.toBean(result2, Map.class);
        //TODO 校验数据是否有？每一个都要去判空
        System.out.println("map = " + map);
        JSONObject data = (JSONObject) map.get("data");
        JSONArray records = (JSONArray) data.get("records");
        List<Post> postList = new ArrayList<>();
        TreeSet tagSet = new TreeSet();

        for (Object record : records) {
            JSONObject tempRecord = (JSONObject) record;
            Post post = new Post();
            post.setTitle(tempRecord.getStr("title"));
            post.setContext(tempRecord.getStr("content"));
            //content
            String content = tempRecord.getStr("content");

            //标签类
            Pattern pattern = Pattern.compile("[A-Za-z]+");
            Matcher matcher = pattern.matcher(content);
            // 进行匹配操作
            String strJava = "(?i)java";
            pattern = Pattern.compile(strJava);
            matcher = pattern.matcher(content);
            if (matcher.find()) {
                String group = matcher.group();
                //String s = group.toUpperCase();
                System.out.println("Match found: " + matcher.group());
                tagSet.add(group);
            }
            String strPython= "(?i)Iterator";
            pattern = Pattern.compile(strPython);
            matcher = pattern.matcher(content);
            if (matcher.find()) {
                String group = matcher.group();
                //String s = group.toUpperCase();
                System.out.println("Match found: " + matcher.group());
                tagSet.add(group);
            }
//            while (matcher.find()) {
//                String group = matcher.group();
//                String java = "(?i)java";
//
//                System.out.println("匹配到的字符串：" + matcher.group());
//            }


            JSONArray tags = (JSONArray) tempRecord.get("tags");
            List<String> tagList = tags.toList(String.class);
            post.setTags(JSONUtil.toJsonStr(tagList));
            postList.add(post);

            System.out.println(postList);
            String lastPrefix = "文章";
            String s1 = JSONUtil.toJsonStr(tagSet);
            System.out.println("s1 = " + s1);
        }
        //System.out.println("postList = " + postList);
    }

```

