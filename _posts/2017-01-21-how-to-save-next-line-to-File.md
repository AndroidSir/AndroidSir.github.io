---
layout: post
title:  "将包含换行等特殊字符的String写进File的解决方法"
date:   2017-01-21
categories: "数据持久化"
tags: "File"
---

* content
{:toc}

**问题发现**：百科类详情模块，第一次从后台获取文字介绍的时候，通过抓包发现含有\r\n换行符，且显示在TextView上能够正确的换行，但是通过如下方法将String写进File，再次使用的时候从File中读取的时候会发现不再有换行效果。




**写文件方法**：

	public static void saveJson(String fileName, String info) {
        try {
            FileOutputStream fos = context.openFileOutput(fileName, Context.MODE_PRIVATE);
            OutputStreamWriter osw = new OutputStreamWriter(fos);
            BufferedWriter bw = new BufferedWriter(osw);
            bw.write(info);
            bw.flush();
            bw.close();
            osw.close();
            fos.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

**读文件方法**：

	public static String getJson(String fileName) {
        String info = null;
        try {
            FileInputStream fis = context.openFileInput(fileName);
            InputStreamReader isr = new InputStreamReader(fis);
            BufferedReader br = new BufferedReader(isr);
            String Line;
            StringBuffer info2 = new StringBuffer();
            while ((Line = br.readLine()) != null) {
                info2.append(Line);
            }
            info = info2.toString();
            br.close();
            isr.close();
            fis.close();

        } catch (Exception e) {
            LogSwitchUtils.logD(FileUtils2.class, "缓存的异常为：" + e);
            return null;
        }
        return info;
    }

通过分析，是因为FileOutputStream、OutputStreamWriter、BufferedWriter等一些流在处理这些转义字符的时候没有将其写入，或者是FileInputStream、InputStreamReader、BufferedReader等没有将其读出。PS：应恶补这些知识。

**问题解决**：

方法1：依然使用上述两个Method，不过在调用之前先对目标String进行编码/解码，如`URLEncoder.encode(schoolDesc)`和`URLDecoder.decode(schoolDescTemp)`；

方法2：将带有特殊字符的String写进SharedPreferences，使用的时候读出；

方法3：使用字节流进行处理,但是该方法有一个不足之处，见读文件方法的注释。方法如下；

**写文件方法**：

	public static void saveStringToFileByByte(Context context, String fileName, String info) {
        try {
            FileOutputStream fos = context.openFileOutput(fileName, Context.MODE_PRIVATE);
            byte[] buf = info.getBytes();
            fos.write(buf);
            fos.close();
        } catch (Exception e) {
            LogSwitchUtils.logE(FileUtils2.class, "fos写缓存时候的异常为：" + e);
            e.printStackTrace();
        }
    }

**读文件方法**：

	public static String getStringFromFileByByte(Context context, String fileName) {
        String result = null;
        StringBuffer sb = new StringBuffer();
        File file = new File(context.getFilesDir().getAbsolutePath() +"/"+ fileName);
        if (!file.exists()) {
            return result;
        }
        try {
            FileInputStream fis = new FileInputStream(file);
            byte[] buf = new byte[1024 * 2];//数组的容量不容易确定，如果给的太大，浪费内容；给的太小，在new String的时候将会发生乱码，因此该方法适用于长度固定的String的File写入及读出
            while (fis.read(buf) > 0) {
                sb.append(new String(buf));
            }
            result = sb.toString();
            fis.close();
        } catch (Exception e) {
            LogSwitchUtils.logE(FileUtils2.class, "fis写缓存时候的异常为：" + e);
            e.printStackTrace();
        }
        return result;
    }

方法4：使用RandomAccessFile进行读写，方法如下：

**写文件方法**：

	public static void saveFileByRaf(Context context, String fileName, String info) {
        File path = new File(context.getFilesDir().getAbsolutePath() + "/baike_detail");
        if (!path.exists()) {
            path.mkdir();
        }
        File file = new File(path, fileName);
        if (file.exists()) {
            file.delete();
        }
        try {
            RandomAccessFile raf = new RandomAccessFile(file, "rw");
            raf.writeUTF(info);
        } catch (Exception e) {
            LogSwitchUtils.logE(FileUtils2.class, "写缓存时候的异常为：" + e);
            e.printStackTrace();
        }
    }

**读文件方法**：

	public static String getFileByRaf(Context context, String fileName) {
        String result = null;
        File path = new File(context.getFilesDir().getAbsolutePath() + "/baike_detail");
        if (!path.exists()) {
            return result;
        };
        File file = new File(path, fileName);
        if (!file.exists()) {
            return result;
        }
        try {
            RandomAccessFile raf = new RandomAccessFile(file, "rw");
            result = raf.readUTF();
        } catch (Exception e) {
            LogSwitchUtils.logE(FileUtils2.class, "读缓存时候的异常为：" + e);
            e.printStackTrace();
        }
        return result;
    }



# Java的IO流

##File

File类是java.io包下代表与平台无关的文件和目录，即不管是文件还是目录都是使用File来操作的，File能新建、删除、重命名文件和目录，但不能访问文件内容本身。

**访问文件名相关的方法**：

1. String getName()：返回此File对象所表示的文件名或路径名（如果是路径，则返回最后一级子路径名）
2. String getPath():返回此File对象所对应的路径名
3. File getAbsoluteFile():返回此File对象的绝对路径（File形式）
4. String getAbsolutePath():返回此File对象所对应的绝对路径（String形式）
5. String getParent():返回此File对象所对应的父目录名
6. boolean renameTo(File newName):重命名此File对象所对应的文件或目录，若成功则返回true

**文件检测相关的方法**：

1. boolean exists():判断File对象所对应的文件或目录是否存在
2. boolean canWrite():判断File对象所对应的文件或目录是否可写
3. boolean canRead():判断File对象所对应的文件或目录是否可读
4. boolean isFile():判断File对象所对应的是否是文件，而不是目录
5. Boolean isDirectory():判断File对象所对应的是否是目录，而不是文件
6. boolean isAbsolute():判断File对象所对应的文件或目录是否是绝对路径

**获取常规文件信息**：

1. long lastModified():返回文件的最后修改时间
2. long length():返回文件内容的长度

**文件操作相关的方法**：

1. boolean createNewFile():当此File对象所对应的文件不存在时，该方法将新建一个该File对象所指定的新文件，如果创建成功则返回true，否则返回false
2. boolean delete():删除File对象所对应的文件或路径
3. void deleteOnExit():注册一个删除钩子，指定当Java虚拟机退出时，删除File对象所对应的文件和目录。
4. static File createTempFile(String prefix,String suffix)
5. static File createTempFile(STring prefix,STring suffix,File directory):创建临时文件

**目录操作相关方法**：

1. boolean mkdir():视图创建一个File对象（目录而非文件）所对应的目录
2. String[] list():列出File对象的所有子文件名和路径名
3. File[] listFiles():列出File对象所有子文件和路径
4. static File[] listRoots():列出系统的所有的根路径

##IO流的分类

1. 输入流和输出流
2. 字节流（InputStream和OutputStream）和字符流(Reader和Writer)
3. 节点流和处理流：直接连接到数据源的是节点流，处理流的使用是为了简单与高效


**访问文件的节点流**：

1. FileInputStream
2. FileReader
3. FileOutputStream
4. FileWiter

这些节点流读和写方法中的容器参数，分别代表读到容器内，将容器内的内容写出去。

**常用的处理流**：

1. 缓冲流：BufferedInputStream、BufferedOutputStream、BufferedReader、BufferedWriter
2. 转换流：InputStreamReader、OutputStreamWriter

##RandomAccessFile

简介：可以直接跳转到文件的任意地方来读写文件，但一个最大的局限是只能读写文件，不能读写其他节点。指针的移动单位是字节。

方法：

1. long getFilePointer():返回文件记录指针的当前位置。
2. void seek(long pos):将文件记录指针定位到pos位置。
3. read（）
4. write()
5. readXxx()
6. writeXxx()
7. mode值："r","rw"
