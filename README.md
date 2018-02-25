# Wecloud开发使用手册 {#zhangyue}

## 架构介绍

正文正文正文正文正文正文

import java.io.\*;

public class a {

```
public static void main\(String\[\] args\) {

    try {

        File f = new File\("E:/a.txt"\);   //a.txt路径

        InputStream is = new FileInputStream\(f\);

        BufferedReader bf = new BufferedReader\(new InputStreamReader\(is\)\);

        String st1 = bf.readLine\(\);

        String st2 = st1.replace\("d", "f"\);

        System.out.println\(st2\);

        File ff = new File\("E:/b.txt"\);    //b.txt路径

        OutputStream os = new FileOutputStream\(ff\);

        BufferedWriter bw = new BufferedWriter\(new OutputStreamWriter\(os\)\);

        bw.write\(st2\);

        bw.flush\(\);

    } catch \(FileNotFoundException e\) {

        // TODO Auto-generated catch block

        e.printStackTrace\(\);

    } catch \(IOException e\) {

        // TODO Auto-generated catch block

        e.printStackTrace\(\);

    }

}
```

}

