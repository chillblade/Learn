
    //Shader在材质面板里的菜单显示的位置
    Shader "CC/Leaf Swing"  
    {
      //材质面板里显示的属性，用来在编辑器里赋值
      Properties {
        _Int ("Int", Int) = 2
        _Float("Float",Float) = 0.3
        _Range("Range",Range(0,5)) = 3.0
        _Color("Color",Color) =(0,0,0,0)
        _Vector("Vector",Vector) =(0,0,0,0)
        //纹理的默认值可以使用内置纹理（White,Black,Gray,Bump）也可以留空，后边花括号的选项5.0后已移除
        _2D("2D",2D) = "White"{}
        _3D("3D",3D) = "Black"{}
        _Cube("Cube",Cube) = ""{}
      }
      //可包含多个SubShader块，Unity会选择第一个可以运行的的Shader
      SubShader
      {
        //渲染状态指令
        LOD 100
        Cull  Back|Front|Off  //剔除模式
        ZTest  Less|Greater|LEqual|GEqual|Equal|NotEqual|Always  深度测试是否通过的条件
        ZWrite  On|Off  深度写入开关
        Blend SrcFactor DstFactor  混合方式
        
        //可选的标签
        Tags
        {
          //
          "RenderType"="Opaque"
          //队列
          "Queue"="Opaque"
        }

        CGPROGRAM
        #pragma surface surf Lambert vertex:vert alpha

        sampler2D _MainTex;
        fixed4 _Pos;
        fixed4 _Direction;
        half _TimeScale;
        half _TimeDelay;
        half _Cutoff;

        struct Input
        {
          half2 uv_MainTex;
        };

        void surf (Input IN, inout SurfaceOutput o)
        {
          half4 c = tex2D (_MainTex, IN.uv_MainTex);
          o.Albedo = c.rgb;
          o.Alpha = c.a;
        }

        void vert (inout appdata_full v)
        {  
          half dis = distance(v.vertex ,_Pos) ;
          half time = (_Time.y + _TimeDelay) * _TimeScale;
          v.vertex.xyz += dis * (sin(time) * cos(time * 2 / 3) + 1) * _Direction.xyz;	//核心，动态顶点变换
        }

        ENDCG
      }

      FallBack "Transparent/Cutout/VertexLit"
    }
