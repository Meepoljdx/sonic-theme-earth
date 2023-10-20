# Sonic-theme-earth

## 说明

该主题的原作者为 [halo-dev](https://github.com/halo-dev)，Halo官方社区；

使用Go Template语法进行改写以适配[Sonic](https://github.com/go-sonic/sonic)，第一次写前端，边搜边改的，有一些已知的问题，比如没有**实现搜索插件**，然后**没法自动解析Markdown目录**。

**模板常用功能经过测试应当是没有问题，如有瑕疵请见谅**

原主题地址：[https://github.com/halo-dev/theme-earth](https://github.com/halo-dev/theme-earth)

评论插件使用了[halo-comment-yu](https://github.com/cetr/halo-comment-yu)插件进行少量修改，原作者为[cetr](https://github.com/cetr)

## 预览截图

演示地址：[meepoljd.top](https://meepoljd.top/)

![image-20231019155243166](https://meepoljd.oss-cn-hangzhou.aliyuncs.com/img/image-20231019155243166.png)

![image-20231019155301991](https://meepoljd.oss-cn-hangzhou.aliyuncs.com/img/image-20231019155301991.png)

![image-20231019155320668](https://meepoljd.oss-cn-hangzhou.aliyuncs.com/img/image-20231019155320668.png)

包含了一个深色模式：

![image-20231019161109635](https://meepoljd.oss-cn-hangzhou.aliyuncs.com/img/image-20231019161109635.png)

## 安装方法

1. [点击](https://github.com/Meepoljdx/sonic-theme-earth/releases/download/1.6.0/sonic-theme-earth-lite.zip)下载无额外功能的包，如果要使用完整版本，需要按照下面的说明进行Sonic代码的修改，然后[点击](https://github.com/Meepoljdx/sonic-theme-earth/releases/download/1.6.0/sonic-theme-earth.zip)下载。
2. 进入后台 -> 外观 -> 主题。
3. 点击右下方按钮选择安装主题，随后选择 `本地上传`。
4. 选择下载好的主题包（zip）即可。

## 说明

侧边栏的功能有两个依赖于新增的模板函数，这个是我自己写的，如果要使用的话，需要自己添加进去：

![image-20231019160937378](https://meepoljd.oss-cn-hangzhou.aliyuncs.com/img/image-20231019160937378.png)

### 添加模板函数方式

**近期文章**

```go
// 在template/extension/post.go中修改

// 添加此方法
func (p *postExtension) addListMostPopularPost() {
	listMostPopularPost := func(top int) ([]*vo.Post, error) {
		ctx := context.Background()
		posts, _, err := p.PostService.Page(ctx, param.PostQuery{
			Page: param.Page{
				PageNum:  0,
				PageSize: top,
			},
			Sort: &param.Sort{
				Fields: []string{"visits,desc"},
			},
			Statuses: []*consts.PostStatus{consts.PostStatusPublished.Ptr()},
		})
		if err != nil {
			return nil, err
		}
		return p.PostAssembler.ConvertToListVO(ctx, posts)
	}
	p.Template.AddFunc("listMostPopularPost", listMostPopularPost)
}

func RegisterPostFunc(template *template.Template, postService service.PostService, postTagService service.PostTagService, postCategoryService service.PostCategoryService, categoryService service.CategoryService, postAssembler assembler.PostAssembler, tagService service.TagService) {
	p := &postExtension{
		Template:            template,
		PostService:         postService,
		PostTagService:      postTagService,
		PostCategoryService: postCategoryService,
		CategoryService:     categoryService,
		PostAssembler:       postAssembler,
		TagService:          tagService,
	}
	p.addListLatestPost()
	p.addGetPostCount()
	p.addGetPostArchiveYear()
	p.addGetPostArchiveMonth()
	p.addListPostByCategoryID()
	p.addListPostByCategorySlug()
	p.addListPostByTagID()
	p.addListPostByTagSlug()
	p.addListMostPopularPost() // 添加此行
}
```

**博客统计**

```go
// 在template/extension/statistic.go中修改，默认没有此文件，需要手动添加

package extension

import (
	"context"

	"github.com/go-sonic/sonic/model/dto"
	"github.com/go-sonic/sonic/service"
	"github.com/go-sonic/sonic/template"
)

type statisticExtension struct {
	Template         *template.Template
	StatisticService service.StatisticService
}

func RegisterStatisticFunc(template *template.Template, statisticService service.StatisticService) {
	s := &statisticExtension{
		Template:         template,
		StatisticService: statisticService,
	}
	s.addGetStatisticsData()
}

func (s *statisticExtension) addGetStatisticsData() {
	getStatisticsDataFunc := func() (*dto.Statistic, error) {
		ctx := context.Background()
		statistic, err := s.StatisticService.Statistic(ctx)
		if err != nil {
			return nil, err
		}
		return statistic, nil
	}
	s.Template.AddFunc("getStatisticsData", getStatisticsDataFunc)
}
```

添加注册逻辑

```go
// 在main.go中添加
		fx.Invoke(
			listener.NewStartListener,
			listener.NewTemplateConfigListener,
			listener.NewLogEventListener,
			listener.NewPostUpdateListener,
			listener.NewCommentListener,
			extension.RegisterCategoryFunc,
			extension.RegisterCommentFunc,
			extension.RegisterTagFunc,
			extension.RegisterMenuFunc,
			extension.RegisterPhotoFunc,
			extension.RegisterLinkFunc,
			extension.RegisterToolFunc,
			extension.RegisterPaginationFunc,
			extension.RegisterPostFunc,
			extension.RegisterStatisticFunc, // 添加这一行
			func(s *handler.Server) {
				s.RegisterRouters()
			},
		),
```

