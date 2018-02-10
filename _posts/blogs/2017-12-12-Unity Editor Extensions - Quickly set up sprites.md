---
layout: page
title: unity编辑器扩展篇-快速设置sprite

---

当导入sprite时，图片总不是我们想要的格式，所以我们每次都需要对Texture的属性进行修改，图片数量多的话的就会特别累，所以写个脚本简化操作岂不是美滋滋
代码很简单，利用TextureImporter修改Texture各项属性，直接上代码，不理解去看我的另一篇博客
http://blog.csdn.net/qq_34244317/article/details/76718284编辑器扩展基础
```
//快速设置精灵
    [MenuItem("Assets/MyEditor/SpriteSet &c")]
    static void SpriteSet()
    {
        //如果选择了对象
        if (Selection.objects.Length > 0)
        {
            //获得所有选中的Texture对象
            foreach (Texture texture in Selection.objects)
            {
                string selectionPath = AssetDatabase.GetAssetPath(texture);
                TextureImporter textureIm = AssetImporter.GetAtPath(selectionPath) as TextureImporter;

                //<-----------------设置参数---------------------->
                textureIm.textureType = TextureImporterType.Sprite;
                textureIm.spriteImportMode = SpriteImportMode.Multiple;
                //<-----------------设置参数---------------------->

                AssetDatabase.ImportAsset(selectionPath);
            }
        }
    }
  
```
  使用前
![这里写图片描述](http://img.blog.csdn.net/20171212005311194?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
使用后
![这里写图片描述](http://img.blog.csdn.net/20171212005322543?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)