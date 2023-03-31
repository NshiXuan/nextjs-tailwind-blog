---
title: 'Java如何将dto的数据映射给po' # 标题
date: '2023/3/31' # 发布时间
# lastmod: '2022/3/10'
tags: [Java] # 标签
draft: false
summary: 'Java如何将dto的数据映射给po'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2ff2a0330b94d4d935cd2cc26a1b4f5~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

在 `Java` 开发中，经常需要将 `Dto` 的数据映射给 `po` ，然后使用 `mybatis-plus` 进行数据库操作保存或更新数据，那么如何将 `Dto` 的数据映射给 `po` 呢？如下

- 如果是添加，通过 `BeanUtils.copyProperties` 将 `dto` 数据拷贝给 `po`

```java
@Transactional
@Override
public CourseBaseInfoDto createCourseBase(Long companyId, AddCourseDto dto) {
    // 1.将dto的数据映射给po courseBaseNew
    CourseBase courseBaseNew = new CourseBase();
    // 一个个get set太麻烦 我们使用BeanUtils.copyProperties把dto的数据拷贝给courseBaseNew
    // copyProperties只要属性名称一致就可以拷贝 如果courseBaseNew中有默认值 dto中没用值 默认值会被覆盖为null
    // 如果我们要set一些数据 防止被覆盖 就写在copyProperties后面
    BeanUtils.copyProperties(dto, courseBaseNew);
    courseBaseNew.setCompanyId(companyId);
    ...

    // 2.插入数据
    int insert = courseBaseMapper.insert(courseBaseNew);
    if (insert <= 0) {
        throw new RuntimeException("添加课程失败");
    }

    ...
}
```

- 如果是更新，从数据库中查询当前信息，再将 `dto` 拷贝给当前信息

```java
@Override
public CourseBaseInfoDto updateCourseBase(Long companyId, EditCourseDto editCourseDto) {
    // 1.参数的合法性校验
    // 判断课程是否存在
    Long courseId = editCourseDto.getId();
    CourseBase courseBase = courseBaseMapper.selectById(courseId);
    if (courseBase == null) {
        MyException.cast("课程不存在");
    }
    // 判断该机构是否为发布该课程的机构
    if (!companyId.equals(courseBase.getCompanyId())) {
        MyException.cast("本机构只修改本机构的课程");
    }

    // 2.封装数据 把最新的editCourseDto数据赋值给courseBase
    BeanUtils.copyProperties(editCourseDto, courseBase);
    // 修改时间
    courseBase.setChangeDate(LocalDateTime.now());

    // 3.修改数据
    int i = courseBaseMapper.updateById(courseBase);
    if (i < 0) {
        MyException.cast("修改课程失败");
    }

    // 4.查询课程信息并返回
    return getCourseBaseInfo(courseId);
}
```
