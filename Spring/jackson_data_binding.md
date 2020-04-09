> REST API 학습 중 현상하나 기억을 위해 메모  

jackson에서 data binding 할때 get으로 정의된 메소드가 호출되어 json으로 응답됨.

```java
// Controller
@GetMapping("")
public ResponsePostsAllDto findAll() {
    return postsService.findAll();
}

// DTO
@Getter
@NoArgsConstructor
public class ResponsePostsAllDto {

    private List<ResponsePostsDto> allPosts;

    public ResponsePostsAllDto(List<Posts> postsList) {
        this.allPosts = postsList.stream()
                .map(posts -> new ResponsePostsDto(posts))
                .collect(Collectors.toList());
    }

    public int size() {
        return this.allPosts.size();
    }

    public List<ResponsePostsDto> getList() {
        return Collections.unmodifiableList(this.allPosts);
    }
}
```

```json
{
    "postsDtoList": [
        {
            "id": 1,
            "uri": "test test",
            "content": "test"
        }
    ],
    "list": [
        {
            "id": 1,
            "uri": "test test",
            "content": "test"
        }
    ]
}
```


```java
// Controller
@GetMapping("")
public ResponsePostsAllDto findAll() {
    return postsService.findAll();
}

// DTO
@Getter
@NoArgsConstructor
public class ResponsePostsAllDto {

    private List<ResponsePostsDto> allPosts;

    public ResponsePostsAllDto(List<Posts> postsList) {
        this.allPosts = postsList.stream()
                .map(posts -> new ResponsePostsDto(posts))
                .collect(Collectors.toList());
    }

    public int size() {
        return this.allPosts.size();
    }

    public List<ResponsePostsDto> findList() {
        return Collections.unmodifiableList(this.allPosts);
    }
}
```

```json
{
    "postsDtoList": [
        {
            "id": 1,
            "uri": "test test",
            "content": "test"
        }
    ]
}
```