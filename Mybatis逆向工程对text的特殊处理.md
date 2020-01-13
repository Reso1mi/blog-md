---
title: 
   Mybatis逆向工程对text的特殊处理
tags: 
  [技术,框架]
date: 2018/5/23 9:00
categories:
		[Web]
---
# Mybatis对Mysql中text类型的特殊处理
- 昨天晚上在做cms的crud的时候遇到的问题，编辑的时候不回显内容，然后f12看了下响应的json里面content也为null，然后去service里面把那个pojo加了toString把几条数据直接打印出来 因为用了分页然后打印出来的也并不是数据

（下次再来探究分页插件） 然后注释掉在打印发现还是没有然后我试了下单独查询，根据主键查询查出来的居然用有！！！这就很奇怪了。。。我当时就以为是byexample的mapper有问题 哈哈哈~~ 其实还是自己框架学的不好的问题

```java
public EasyUIJsonResult listContent(long categoryId,int page,int rows) {
		EasyUIJsonResult result=new EasyUIJsonResult();
		
		//测试过程中发现PageHelper的一些问题 等下再来研究
		//PageHelper.startPage(page, rows);
		TbContentExample example =new TbContentExample();
		example.createCriteria().andCategoryIdEqualTo(categoryId);
		List<TbContent> contents = dao.selectByExample(example);
		PageInfo<TbContent> info=new PageInfo<>(contents);
	
		//加上PageHelper后这里打印出来的就不是contents对象了就是Page对象
		//而且查出来的数据里面content为null (数据库中有 猜想可能是逆向工程有问题);
		System.out.println(contents);
		
		//测试下单独查询 主键查询没问题 取的到content
		System.out.println(dao.selectByPrimaryKey((long)42));
		
		//单独测试下只查一个    也没有
		TbContentExample example2 =new TbContentExample();
		Criteria criteria2 = example2.createCriteria();
		criteria2.andCategoryIdEqualTo((long)90);
		System.out.println(dao.selectByExample(example2 ));
		
		result.setTotal(info.getTotal());
		result.setRows(contents);
		return result;
	}
```
- 然后就面向百度了一波，还真就查到了遇到跟我一样的问题[同道中人](https://ask.csdn.net/questions/205320)
selectByExampleWithBLOBs用这个方法就可以了 看名字就知道更这个有关哈哈哈
然后就愉快的解决了![image](http://p1.cdn.img9.top/ipfs/QmVhiMUQoFbiHc8BJxFtSzCUxVrZffnj9vYD7yM5YmCjGL?1.png)
- 不仅仅是select update也有这个方法
```java
public interface TbContentMapper {
    int countByExample(TbContentExample example);

    int deleteByExample(TbContentExample example);

    int deleteByPrimaryKey(Long id);

    int insert(TbContent record);

    int insertSelective(TbContent record);

    List<TbContent> selectByExampleWithBLOBs(TbContentExample example);

    List<TbContent> selectByExample(TbContentExample example);

    TbContent selectByPrimaryKey(Long id);

    int updateByExampleSelective(@Param("record") TbContent record, @Param("example") TbContentExample example);

    int updateByExampleWithBLOBs(@Param("record") TbContent record, @Param("example") TbContentExample example);

    int updateByExample(@Param("record") TbContent record, @Param("example") TbContentExample example);

    int updateByPrimaryKeySelective(TbContent record);

    int updateByPrimaryKeyWithBLOBs(TbContent record);

    int updateByPrimaryKey(TbContent record);
}
```
