# 스프링에서 REST 설계 후 제네릭 타입 객체 테스트하기

사용 가능한 것은 **new ParameterizeTypeReference**이다.  

아래의 소스 코드를 봐보자.
CommonResponse를 정의 후 상속받은 SuccessResponse, ErrorResponse가 있다.  
해당 객체의 사용은 DTO를 갖는 부분이다.  

DTO는 여러 타입으로 정의되기 때문에 어떤게 올지 모른다고 가정했다.  
Controller에서는 ResponseEntity로 반환을 하며 ResponseService에서 Success, Error 2가지를 분리하여 처리한다.  

제네릭 타입으로 정의를하여 범용적으로 사용 가능하도록 설계를 했다.  

여기서 반환에는 문제가 없었다. 하지만 Test코드 작성 시 최초 getForEntity로 ResponseEntity를 반환받도록 구현을 한 상태에서 제네릭타입의 DTO를 받을때 Object 타입을 DTO로 cast에 문제가 있었다.  
변경된 코드는 exchange를 사용하여 Http Method GET으로 정의하여 **ParameterizeTypeReference**를 활용하여 제네릭 타입을 맞춰주어 테스트 코드를 작성하였다.  

```java
@Getter
@Setter
public class CommonResponseDto {
    private boolean result;
    private int code;
    private String message;
}

@Getter
@Setter
public class SuccessResponseDto<T> extends CommonResponseDto {
    private T response;
}

@Getter
@Setter
public class ErrorResponseDto extends CommonResponseDto {

}
```

```java
@Service
public class ResponseService {
    public ErrorResponseDto getErrorResponse(ResponseEnums responseEnums) {
        boolean result = false;
        if (ResponseEnums.NOT_FOUND.equals(responseEnums)) {
            result = true;
        }
        ErrorResponseDto returnObj = new ErrorResponseDto();
        setCommonResponseData(responseEnums, returnObj, result);
        return returnObj;
    }

    public <T> SuccessResponseDto getSuccessResponse(ResponseEnums responseEnums, T dto) {
        SuccessResponseDto<T> returnObj = new SuccessResponseDto<>();
        setCommonResponseData(responseEnums, returnObj, true);
        returnObj.setResponse(dto);
        return returnObj;
    }

    private <T extends CommonResponseDto> void setCommonResponseData(ResponseEnums responseEnums, T returnObj, boolean result) {
        returnObj.setCode(responseEnums.getCode());
        returnObj.setMessage(responseEnums.getMessage());
        returnObj.setResult(result);
    }
}
```


```java
@GetMapping("")
public ResponseEntity findAll() {
    // .. 비지니스 로직
    return ResponseEntity.ok(responseService.getSuccessResponse(ResponseEnums.OK, dto));
}
```

```java
@Test
public void 전체조회() throws Exception {
    // given
    List<RegionInfo> regionInfos = new ArrayList<>();
    for (int i = 0; i < 3; i++) {
        regionInfos.add(getRegionInfoSampleData(i));
    }
    String uri = "http://localhost:" + port + "/api/v1/region";
    RegionInfo compareRegionInfo = regionInfos.get(0);

    // when
    ResponseEntity<SuccessResponseDto<ListRegionInfoResponseDto>> responseEntity = restTemplate.exchange(
            uri,
            HttpMethod.GET,
            null,
            new ParameterizedTypeReference<SuccessResponseDto<ListRegionInfoResponseDto>>() {});
    RegionInfoResponseDto responseRegionInfo = responseEntity
            .getBody()
            .getResponse()
            .getRegionInfoResponseDtos()
            .get(0);

    // then
    assertAll(
            () -> assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK),
            () -> assertThat(responseEntity.getHeaders().getContentType()).isEqualTo(MediaType.APPLICATION_JSON),
            () -> assertThat(responseRegionInfo.getId()).isEqualTo(compareRegionInfo.getId()),
            () -> assertThat(responseRegionInfo.getRegionCode()).isEqualTo(compareRegionInfo.getRegion().getCode()),
            () -> assertThat(responseRegionInfo.getRegionName()).isEqualTo(compareRegionInfo.getRegion().getName()),
            () -> assertThat(responseRegionInfo.getLimit()).isEqualTo(LimitKeywordEnums.convertLimitNumberToString(compareRegionInfo.getLimit())),
            () -> assertThat(responseRegionInfo.getUsage()).isEqualTo(compareRegionInfo.getUsage()),
            () -> assertThat(responseRegionInfo.getTarget()).isEqualTo(compareRegionInfo.getTarget()),
            () -> assertThat(responseRegionInfo.getInstitute()).isEqualTo(compareRegionInfo.getInstitute()),
            () -> assertThat(responseRegionInfo.getMgmt()).isEqualTo(compareRegionInfo.getMgmt()),
            () -> assertThat(responseRegionInfo.getRate()).isEqualTo(compareRegionInfo.getRate()),
            () -> assertThat(responseRegionInfo.getReception()).isEqualTo(compareRegionInfo.getReception())
    );
}
```