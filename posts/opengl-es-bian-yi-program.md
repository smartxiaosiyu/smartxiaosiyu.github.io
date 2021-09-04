---
title: 'OpenGL ES 初始化着色器程序'
date: 2021-09-04 11:45:43
tags: [OpenGL]
published: true
hideInList: false
feature: 
isTop: false
---
### 1.获取着色器program
 +  1.编译顶点、片元着色器
     + 1.编译shader代码
        - 1.获取shader 路径
    `NSString *shaderPath = [[NSBundle mainBundle] pathForResource:name ofType:shaderType == GL_VERTEX_SHADER ? @"vsh" : @"fsh"];`
    `NSError *error;`
    `NSString *shaderString = [NSString stringWithContentsOfFile:shaderPath encoding:NSUTF8StringEncoding error:&error];`
        - 2. 创建shader->根据shaderType
    `GLuint shader = glCreateShader(shaderType)`
        - 3.获取shader source
    `glShaderSource(shader, 1, &shaderStringUTF8, &shaderStringLength);`
        - 4.编译shader
    `glCompileShader(shader);`
         - 5.返回shader
    ` return shader;`

    + 2.将顶点/片元附着到program
    `GLuint program = glCreateProgram();`
    `glAttachShader(program, vertexShader);`
    `glAttachShader(program, fragmentShader);`
    + 3.linkProgram
    `glLinkProgram(program);`
    + 4.返回program
    `return program;`
+ 2.use Program
   ` glUseProgram(program);`
+ 3.获取Position,Texture,TextureCoords 的索引位置
    `GLuint positionSlot = glGetAttribLocation(program, "Position");`
    `GLuint textureSlot = glGetUniformLocation(program, "Texture");`
    `GLuint textureCoordsSlot = glGetAttribLocation(program, "TextureCoords");`纹理坐标
+ 4.激活纹理,绑定纹理ID
    `glActiveTexture(GL_TEXTURE0);`
    `glBindTexture(GL_TEXTURE_2D, self.textureID);`
+ 5.将纹理id传入
  `glUniform1i(textureSlot, 0);`
+ 6.打开positionSlot 属性并且传递数据到positionSlot中(顶点坐标)
    `glEnableVertexAttribArray(positionSlot);`
    `glVertexAttribPointer(positionSlot, 3, GL_FLOAT, GL_FALSE, sizeof(SenceVertex), NULL + offsetof(SenceVertex, positionCoord));`
+ 7.保存program,界面销毁则释放
    `self.program = program;`