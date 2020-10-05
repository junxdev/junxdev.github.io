---
title: "MyBatis: 복수의 파라미터를 전달하는 오버로드 메소드를 mapper에서 조건문으로 처리"
excerpt: "BindingException: Parameter not found. `@Param` Annotation으로 파라미터를 제공하면 Map 개체인 _parameter가 생성되므로 containsKey 메소드를 활용해 해결"
tags: [MyBatis, Spring, Java]
---
* toc
{:toc}

## 1. 문제
환경: VMWare Workstation 15 Player, Windows 10, Spring Tool Suite 3, Oracle 11g XE, MyBatis 3.5.5, Spring 4.3.22


* 조건에 따라 데이터 목록을 불러오는 기능을 개발 중
* 메소드를 오버로드해봄(그냥 해보고 싶었음)
* 코드 예시는 다음과 같음
* DAO interface

``` java
public interface ItemDao {
    List<ItemVo> getItemList();
    List<ItemVo> getItemList(@Param("color") String color);
    List<ItemVo> getItemList(@Param("color") String color, @Param("size") String size);
}
```

* Mapper.xml

``` xml
<mapper>
    <select resultType="com.example.shop.item.model.entity.ItemVo">
        SELECT * FROM item
        <where>
            <if test="color != null">
                color = #{color}
            </if>
            <if test="size != null">
                AND size = #{size}
            </if>
        </where>
    </select>
</mapper>
```

* JUnit

``` java
public class ItemTest {
    @Test
	public void getItemList() {
		ItemDao dao = sqlSession.getMapper(ItemDao.class);
		List<ItemVo> list1 = dao.selectAll(); // Test 1
		List<ItemVo> list2 = dao.selectAll("black"); // Test 2
		List<ItemVo> list3 = dao.selectAll("black", "XL"); // Test 3
	}
}
```

* 위 테스트 코드에서 Test 2는 아래의 에러로 실패

``` 
org.apache.ibatis.binding.BindingException: Parameter 'size' not found. Available parameters are [color, param1]
```

* 구글링하며 if 조건에 `_parameter != null`, `_parameter.equals('size')` 등을 넣었지만 같은 에러로 실패
(남들은 대부분 된다고 함... 개체에 담아 보낸 건가?)

``` xml
<mapper>
    <select resultType="com.example.shop.item.model.entity.ItemVo">
        SELECT * FROM item
        <where>
            <if test="color != null">
                color = #{color}
            </if>
            <if test="_parameter != null AND _parameter.equals('size')">
                AND size = #{size}
            </if>
        </where>
    </select>
</mapper>
```


## 2. 원인
* `@Param` Annotation으로 파라미터를 제공하면 Test 2 기준 `{color=black, param1=black}`를 가진 Map 개체 `_parameter`가 생성
* 아래 코드로 출력해보면 알 수 있음

``` xml
<bind name="param" value="_parameter.toString()"></bind>
SELECT * FROM item WHERE color = #{param}
```

* 따라서 Map 개체에 `equals(String str)` 메소드를 사용해서 실패(Map 인터페이스는 `eqauls(Object obj)` 메소드를 보유)


## 3. containsKey(String str)로 해결
* Map 개체가 특정 문자열의 Key를 갖고 있는지 궁금한 것이므로 `containsKey(String str)` 메소드 사용
* Mapper.xml을 수정

``` xml
<mapper>
    <select resultType="com.example.shop.item.model.entity.ItemVo">
        SELECT * FROM item
        <where>
            <if test="_parameter.containsKey('color')">
                color = #{color}
            </if>
            <if test="_parameter.containsKey('size')">
                AND size = #{size}
            </if>
        </where>
    </select>
</mapper>
```

* 당장은 작동하나 미래에 발생할 일은 아직 잘 모르겠음
* 애초에 이렇게 설계했으면 안 됐음