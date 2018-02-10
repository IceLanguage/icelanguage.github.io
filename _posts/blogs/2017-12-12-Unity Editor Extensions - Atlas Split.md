---
layout: page
title: unity编辑器扩展篇-图集拆分
category: 
    - blogs
---


要写一个属于自己的游戏demo，需要大量的美术资源和音频资源等等，但网上找到的美术资源大部分都是NGUI的图集，我希望能把图集拆开，变成一个个小小的Sprite。
![这里写图片描述](http://img.blog.csdn.net/20171212195532485?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
代码如下

```
//导出单个精灵
    [MenuItem("Assets/MyEditor/SingleSpritesExport &x")]
    static void SingleSpritesExport()
    {
        string resourcesPath = "Assets/Resources/";
        foreach (Object obj in Selection.objects)
        {
            string selectionPath = AssetDatabase.GetAssetPath(obj);

            // 必须最上级是"Assets/Resources/"
            if (selectionPath.StartsWith(resourcesPath))
            {
                string selectionExt = System.IO.Path.GetExtension(selectionPath);
                if (selectionExt.Length == 0)
                {
                    continue;
                }

                // 得到导出路径
                string loadPath = selectionPath.Remove(selectionPath.Length - selectionExt.Length);
                loadPath = loadPath.Substring(resourcesPath.Length);

                // 加载此文件下的所有资源
                Sprite[] sprites = Resources.LoadAll<Sprite>(loadPath);
                if (sprites.Length > 0)
                {
                    // 创建导出文件夹
                    string outPath = Application.dataPath + "/outSprite/" + loadPath;
                    System.IO.Directory.CreateDirectory(outPath);

                    foreach (Sprite sprite in sprites)
                    {
                        // 创建单独的纹理
                        Texture2D tex = new Texture2D((int)sprite.rect.width, (int)sprite.rect.height, sprite.texture.format, false);
                        tex.SetPixels(sprite.texture.GetPixels((int)sprite.rect.xMin, (int)sprite.rect.yMin,
                            (int)sprite.rect.width, (int)sprite.rect.height));
                        tex.Apply();

                        // 写入成PNG文件
                        System.IO.File.WriteAllBytes(outPath + "/" + sprite.name + ".png", tex.EncodeToPNG());
                    }
                    Debug.Log(string.Format("Export {0} to {1}", loadPath, outPath));
                }
            }
        }
        Debug.Log("Export All Sprites Finished");
    }
```
注意要把图集放到Resources文件下并勾选Read/Write Enable
![这里写图片描述](http://img.blog.csdn.net/20171212202005972?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

拆分后
![这里写图片描述](http://img.blog.csdn.net/20171212200409838?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)