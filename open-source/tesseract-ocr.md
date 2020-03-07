# Tesseract OCR

Tesseract OCR是一個開源OCR Opensource，可針對各國語言進行OCR

若條件算不複雜的需求時，可以使用這個套件解決OCR問題

{% embed url="https://github.com/tesseract-ocr/tesseract" %}

## 1. Concept / Request

需求：老舊設備中機台電腦上，操作程式中的參數取得

限制：該程式無法將相關參數拋送

![](../.gitbook/assets/image%20%28349%29.png)

作法：在此限制下，執行概念如下

![](../.gitbook/assets/image%20%28471%29.png)

## 2. Develop

方便示意，假設機台程式UI如下，需要抓取的值為紅框所示

![](../.gitbook/assets/image%20%2874%29.png)

To Do List：

* [x] 提供設定圖片要抓取位置的功能\(坐標、名稱\) =&gt; Labeling
* [x] 將每一個設定好的位置之影像截取出來並進行一定程度處理 =&gt; Preprocess
* [x] 針對每一個處理過的圖片進行OCR辨示圖片中的文字 =&gt; OCR

Visual Studio C\#為例進行程式開發

### 2-1. Labeling

Labeling部分需要的功能有：

* [x] 讀取圖片並顯示在操作界面中
* [x] 框選範圍並可以針對該範圍給予一個變數名稱
* [x] 儲存所有選取設定值

Visual Studio \(2017/2019\)建立新專案

選擇Windows Forms APP \(.Net Framework\)

![](../.gitbook/assets/image%20%28148%29.png)

![](../.gitbook/assets/image%20%28414%29.png)

畫面左方中加入一個Button - Load Image：為了選取圖片來源

![](../.gitbook/assets/image%20%2846%29.png)

畫面右方新增一個pictureBox：顯示圖片用，因為圖片大小可能不會等於PictureBox大小，所以Size Mode要選擇Zoom，讓程式自動縮放圖片

![](../.gitbook/assets/image%20%28314%29.png)

![](../.gitbook/assets/image%20%28229%29.png)

Button LoadImage Click事件處理

![](../.gitbook/assets/image%20%28484%29.png)

開啟FileDialog，並限制可選擇的副檔名為bmp / jpg / png

開啟後存成bitmap後，將PictureBox1的Image指向這個圖檔

註：轉成bitmap後，img變數要dispose，免得造成memoery leak、file lock

```csharp
        private void btnLoadImage_Click(object sender, EventArgs e)
        {
            string sFileName = "";

            try
            {
                OpenFileDialog fileDialog = new OpenFileDialog();
                //filter
                fileDialog.Filter = "圖片|*.bmp;*.jpg;*.png";
                //show folder dialog
                if (fileDialog.ShowDialog() != DialogResult.OK)
                {
                    return;
                }

                sFileName = fileDialog.FileName;

                Image img = Image.FromFile(sFileName);
                Bitmap bitmap = new Bitmap(img);
                pictureBox1.Image = bitmap;

                img.Dispose();
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }
```

測試點選Button Load Image，可以成功跳出檔案選擇視窗

![](../.gitbook/assets/image%20%28331%29.png)

選取檔案後，顯示在右方pictureBox中





![](../.gitbook/assets/image%20%28186%29.png)

接著為了要做到讓使用者框選一個區塊的功能\(例如小畫家中的框選功能\)

![](../.gitbook/assets/image%20%2867%29.png)

針對PictureBox1中的Mouse Down / Up / Move事件進行處理，並新增幾個變數紀錄需要的座標\(開始座標\(左上XY\)、結束座標\(右下XY\)\)

![](../.gitbook/assets/image%20%28228%29.png)

其中Refresh\(\)可以觸發整個FORM重新繪製，因此搭配pictureBox1的Paint事件可以將紅框畫出

```csharp
         private void pictureBox1_MouseDown(object sender, MouseEventArgs e)
        {
            
            if (pictureBox1.Image == null) { return; }
            Refresh();
            pLeftUpper = new Point();
            pRightDown = new Point();
            isMouseDown = true;
            pLeftUpper = e.Location;
            
        }


        private void pictureBox1_MouseUp(object sender, MouseEventArgs e)
        {
            if (isMouseDown)
            {
                pRightDown = e.Location;
                isMouseDown = false;
            }
        }

        private void pictureBox1_MouseMove(object sender, MouseEventArgs e)
        {
            if (pictureBox1.Image == null) { return; }
            if (isMouseDown == true)
            {
                pRightDown = e.Location;
                Refresh();
            }
        }

        private void pictureBox1_Paint(object sender, PaintEventArgs e)
        {
            if (rect_selected != null)
            {
                rect_selected = new Rectangle();
                rect_selected.X = Math.Min(pLeftUpper.X, pRightDown.X);
                rect_selected.Y = Math.Min(pLeftUpper.Y, pRightDown.Y);
                rect_selected.Width = Math.Abs(pLeftUpper.X - pRightDown.X);
                rect_selected.Height = Math.Abs(pLeftUpper.Y - pRightDown.Y);
                e.Graphics.DrawRectangle(Pens.Red, rect_selected);
            }
        }
```

![](../.gitbook/assets/image%20%2883%29.png)

## 

測試OK，可以依照選取的位置進行框選動作

![](../.gitbook/assets/image%20%28348%29.png)

為方便顯示座標位置，新增一個richtextbox

![](../.gitbook/assets/image%20%28163%29.png)

加入方法回報座標\(傳入point\)並在mouse down / up / move事件中呼叫回報方法\(ReportLocation\)

回報位置，傳入Point，顯示X、Y座標

```csharp
        private void ReportLoaction(Point location1)
        {
            richTextBox1.Text = $"X={location1.X}\r\nY={location1.Y}";
        }
```

Mouse事件處理：框選位置\(pLeftUpper、pRightDown要Reset\)、Refresh\(\)會觸發畫面重繪

```csharp
  private void pictureBox1_MouseDown(object sender, MouseEventArgs e)
        {
            
            if (pictureBox1.Image == null) { return; }
            Refresh();
            pLeftUpper = new Point();
            pRightDown = new Point();
            isMouseDown = true;
            pLeftUpper = e.Location;

            ReportLoaction(e.Location); //回報座標
        }


        private void pictureBox1_MouseUp(object sender, MouseEventArgs e)
        {
            if (isMouseDown)
            {
                pRightDown = e.Location;
                ReportLoaction(e.Location); //回報座標
                isMouseDown = false;
            }
        }

        private void pictureBox1_MouseMove(object sender, MouseEventArgs e)
        {
            if (pictureBox1.Image == null) { return; }
            if (isMouseDown == true)
            {
                pRightDown = e.Location;
                ReportLoaction(e.Location); //回報座標
                Refresh();
            }
        }
```

測試可以正常顯示座標位置

![](../.gitbook/assets/image%20%28462%29.png)

新增方法ReportAllLocation方法顯示左上、右下座標並在Mouse UP事件中呼叫此方法\(Mouse Up為框選結束時間點\)

```csharp
 private void ReportAllLoaction(Point location1,Point location2)
        {
            richTextBox1.Text = $"X1={location1.X}\r\nY1={location1.Y}\r\n";
            richTextBox1.Text += $"X2={location2.X}\r\nY2={location2.Y}";
        }
```

```csharp
      private void pictureBox1_MouseUp(object sender, MouseEventArgs e)
        {
            if (isMouseDown)
            {
                pRightDown = e.Location;
                //ReportLoaction(e.Location); //回報座標
                ReportAllLoaction(pLeftUpper, pRightDown); //左上、右下
                isMouseDown = false;
            }
        }
```

測試正常

![](../.gitbook/assets/image%20%28263%29.png)

因為PictureBox是整個右半邊，因此X,Y起始點會在藍色標記位置，但圖片因為等比例縮放關係，不會對齊左上角，圖片的X、Y起始位置應為綠色框選位置

![](../.gitbook/assets/image%20%28141%29.png)

![](../.gitbook/assets/image%20%28100%29.png)

為了解決縮放問題，新增GetActualLocation方法，以取得圖片中真實位置

```csharp
        private Point GetActualLocation(Point location1)
        {
            Int32 mouseX = location1.X;
            Int32 mouseY = location1.Y;
            //圖片實際大小
            Int32 realW = pictureBox1.Image.Width;
            Int32 realH = pictureBox1.Image.Height;
            //目前畫面上的大小
            Int32 currentW = pictureBox1.ClientRectangle.Width;
            Int32 currentH = pictureBox1.ClientRectangle.Height;
            //計算縮放比例
            Double zoomW = (currentW / (Double)realW);
            Double zoomH = (currentH / (Double)realH);
            Double zoomActual = Math.Min(zoomW, zoomH);
            //依縮放換算真實座標
            Double padX = zoomActual == zoomW ? 0 : (currentW - (zoomActual * realW)) / 2;
            Double padY = zoomActual == zoomH ? 0 : (currentH - (zoomActual * realH)) / 2;
            Int32 realX = (Int32)((mouseX - padX) / zoomActual);
            Int32 realY = (Int32)((mouseY - padY) / zoomActual);
            int x = realX < 0 || realX > realW ? -1 : realX;
            int y = realY < 0 || realY > realH ? -1 : realY;
            return new Point(x, y);
        }
```

```csharp
 private void ReportAllLoaction(Point location1,Point location2)
        {
            richTextBox1.Text = $"X1={location1.X}\r\nY1={location1.Y}\r\n";
            richTextBox1.Text += $"X2={location2.X}\r\nY2={location2.Y}";

            //取得真實座標
            Point realLocation1 = GetActualLocation(location1);
            Point realLocation2 = GetActualLocation(location2);
            richTextBox1.Text += "\r\n Real Location：\r\n";
            richTextBox1.Text += $"X1={realLocation1.X}\r\nY1={realLocation1.Y}\r\n";
            richTextBox1.Text += $"X2={realLocation2.X}\r\nY2={realLocation2.Y}";
        }
```

![](../.gitbook/assets/image%20%28391%29.png)

解決座標問題後，接著新增DataGridView，以顯示、記錄每一個框選的Label名稱、座標資料

![](../.gitbook/assets/image%20%28341%29.png)

新增DataTable dtLabels並指定GridView的DataSource為dtLabels

![](../.gitbook/assets/image%20%28293%29.png)



![](../.gitbook/assets/image%20%28157%29.png)

Label輸入新增Enter事件處理 - ttbLabel\_KeyDown

![](../.gitbook/assets/image%20%28360%29.png)

因需取得實際在圖片上的座標位置\(非ui上的\)，因此需Call GetActualLocation\(\)

```csharp
        private void ttbLabel_KeyDown(object sender, KeyEventArgs e)
        {
            try
            {
                if(e.KeyCode != Keys.Enter) { return; }

                string sLabelName = ttbLabel.Text.Trim();

                if(sLabelName == "")
                {
                    throw new Exception("LabelName不可為空!!");
                }

                Point location1 = GetActualLocation(pLeftUpper);
                Point location2 = GetActualLocation(pRightDown);

                DataRow dr_new = dtLabels.NewRow();
                dr_new["LABEL_NAME"] = sLabelName;
                dr_new["X1"] = location1.X;
                dr_new["Y1"] = location1.Y;
                dr_new["X2"] = location2.X;
                dr_new["Y2"] = location2.Y;

                dtLabels.Rows.Add(dr_new);

                ttbLabel.Text = "";
                ttbLabel.Focus();

            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }
```

![](../.gitbook/assets/image%20%28303%29.png)

### 2-2. Image preprocess

* [x] 讀取Label設定 - 取得座標位置並將對應圖片取出
* [x] 圖片處理 - 將圖片進行前處理，以利後續OCR使用

新增GetTagged Image Button

![](../.gitbook/assets/image%20%28375%29.png)

為了方便顯示Tagged的圖片，因此在gvLabel中新增Image欄位\(為了顯示圖片\)

![](../.gitbook/assets/image%20%28481%29.png)

ImageLayout要調整為Zoom

![](../.gitbook/assets/image%20%28416%29.png)

為了進行影像處理，使AForge.Imaging套件，進行圖片處理

![](../.gitbook/assets/image%20%2836%29.png)

新增ImageProcessHelper類別，裡面實作AForge方法

![](../.gitbook/assets/image%20%28446%29.png)

因為需要將圖片中特定位置截取圖片，新增Crop方法

![](../.gitbook/assets/image%20%28115%29.png)

dtLabels新增IMAGE欄位，以記錄圖片

![](../.gitbook/assets/image%20%28232%29.png)

GetTaggedImage Click事件，宣告告ImageHelper，針對每一個Tag好的座標進行Crop動作並存入IMAGE欄位

![](../.gitbook/assets/image%20%28308%29.png)

測試框選兩個位欄位，按下GetTaggetImage前，圖示為空

![](../.gitbook/assets/image%20%28430%29.png)

點選Get Tagged Image後，將圖片顯示在image欄位中

![](../.gitbook/assets/image%20%28297%29.png)

在進行ocr前，針對圖片需要進行特別的前處理，一般最常見為放大、灰階、絕對黑白，以增加辨示度\(因為ocr model訓練時，通常是用絕對黑白或灰階圖片進行訓練\)

新增img資料夾

![](../.gitbook/assets/image%20%28246%29.png)

新增幾個圖片前處理動作

![](../.gitbook/assets/image%20%28350%29.png)

```csharp
 private void btnGetTaggedImage_Click(object sender, EventArgs e)
        {
            try
            {
                if(dtLabels.Rows.Count == 0)
                {
                    return; //
                }
                string sImgPath = @".\img\";
                ImageHelper ih = new ImageHelper();
                string sLabelName = "";
                Bitmap source = new Bitmap(pictureBox1.Image);
                foreach(DataRow dr_label in dtLabels.Rows)
                {
                    sLabelName = dr_label["LABEL_NAME"].ToString();
                    Point p1 = new Point((int)dr_label["X1"], (int)dr_label["Y1"]);
                    Point p2 = new Point((int)dr_label["X2"], (int)dr_label["Y2"]);

                    //抓取標記的範圍
                    Bitmap img = ih.Crop(source, p1,p2);
                    img.Save($"{sImgPath}{sLabelName}-Step1-Ori.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //放大五倍
                    img = ih.Resize(img, img.Width * 5, img.Height * 5);
                    img.Save($"{sImgPath}{sLabelName}-Step2-Resize2.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //轉成灰階
                    img = ih.SetGrayscale(img);
                    img.Save($"{sImgPath}{sLabelName}-Step3-SetGrayscale.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //高斯模糊(為了解決文字解析度或字型造成缺口問題)
                    img = ih.GaussianBlur(img);
                    img.Save($"{sImgPath}{sLabelName}-Step4-GaussianBlur.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //轉成絕對黑白
                    img = ih.SetToBW(img, 190);
                    img.Save($"{sImgPath}{sLabelName}-Step5-ConvertTo1Bpp1.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //更新Image欄位
                    dr_label.BeginEdit();
                    dr_label["IMAGE"] = img;
                    dr_label.EndEdit();
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }
```

ImageHelper類別中新增圖片處理方法，\(可引用Aforge中的方法\)

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using AForge.Imaging;
using AForge.Imaging.Filters;

namespace TesseractDemo
{
    public class ImageHelper
    {

        public Bitmap Crop(Bitmap bmp, Point p1,Point p2)
        {

            Point location = new Point(Math.Min(p1.X, p2.X), Math.Min(p1.Y, p2.Y));
            Size size = new Size(Math.Abs(p1.X - p2.X) + 1, Math.Abs(p1.Y - p2.Y) + 1);

            Rectangle rectangle = new Rectangle(location, size);

            // create filter
            Crop filter = new Crop(rectangle);
            // apply the filter
            Bitmap newImage = filter.Apply(bmp);

            return newImage;
        }

        public Bitmap Resize(Bitmap bmp, int newWidth, int newHeight)
        {

            // create filter
            ResizeNearestNeighbor filter = new ResizeNearestNeighbor(newWidth, newHeight);
            // apply the filter
            Bitmap newImage = filter.Apply(bmp);

            return newImage;
        }

        //SetGrayscale
        public Bitmap SetGrayscale(Bitmap img)
        {

            // create grayscale filter (BT709)
            Grayscale filter = new Grayscale(0.2125, 0.7154, 0.0721);
            // apply the filter
            Bitmap grayImage = filter.Apply(img);

            return grayImage;
        }


        public Bitmap GaussianBlur(Bitmap bSource)
        {
            GaussianBlur gaussianBlur = new GaussianBlur(4, 11);//11

            Bitmap bTemp = gaussianBlur.Apply(bSource);

            return bTemp;
        }

        public Bitmap SetToBW(Bitmap bmp, int ithreshold)
        {
            // create filter
            Threshold filter = new Threshold(ithreshold);
            // apply the filter
            filter.ApplyInPlace(bmp);
            return bmp;
        }
    }
}

```

### 2-3. OCR\(Tesseract\)

* [x] 載入Tesseract OCR套件
* [x] 新增OCRHelper類別
* [x] 新增方法 - 傳入圖片並回傳OCR結果

將圖片處理完畢後，需要將圖片傳入具OCR功能的MODEL，.Net C\#常見的為Tesseract OCR \(由Google團隊發起\)

在專案中點選Nuget套件管理，查詢Tesseract，選擇Tesseract套件進行安裝

![](../.gitbook/assets/image%20%283%29.png)

安裝完套件後，會新增x64、x84資料夾\(包含OCR Engine\)、Tesseract.dll

![](../.gitbook/assets/image%20%28320%29.png)

下載訓練好的OCR Model

{% embed url="https://tesseract-ocr.github.io/tessdoc/Data-Files" %}

選擇English語言的Model

![](../.gitbook/assets/image%20%28168%29.png)

![](../.gitbook/assets/image%20%2813%29.png)

下載完成後，專案執行檔根目錄下，建立tessdata資料夾\(名稱一定要是tessdata\)，將下載的檔案中eng.traineddata放在該資料下

![](../.gitbook/assets/image%20%28220%29.png)

建立OCRHelper類別

![](../.gitbook/assets/image%20%28122%29.png)

在OCRHelper類別中，建立一個STATIC方法\(這樣才不需要new OCRHelper物件就可以直接使用該方法\)

Tesseract OCR使用方法很簡單，1. 宣告Engine 2. 宣告Page並圖片傳入 3. 呼叫Page.GetText方法即可

其中因為測試對象都是數字，所以可以指定OCR Engine判斷文字時，從0~9中挑選最有可能者

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.IO;
using Tesseract;
using System.Drawing.Drawing2D;
using System.Drawing.Imaging;
using System.Drawing;
using System.Windows.Forms;

namespace TesseractDemo
{
    public class OCRHelper
    {
        public static string OCR(Bitmap img)
        {
            TesseractEngine ocr = null;
            string sResult = "";
            try
            {
                ocr = new TesseractEngine("./tessdata", "eng"); //初始化 (一定要放在tessdata資料夾下)
                ocr.SetVariable("tessedit_char_whitelist", "0123456789"); //強迫Char List，較準確

                Page page = ocr.Process(img, PageSegMode.SingleLine);
                sResult = page.GetText();//result
                page.Dispose();
            }
            catch (Exception ex)
            {
                //MessageBox.Show(ex.Message);
                sResult = "";
            }
            finally
            {
                ocr?.Dispose();
            }
            return sResult.Replace(" ", "");
        }
    }
}

```

![](../.gitbook/assets/image%20%2876%29.png)

![](../.gitbook/assets/image%20%286%29.png)



![](../.gitbook/assets/image%20%2862%29.png)

![](../.gitbook/assets/image%20%288%29.png)

![](../.gitbook/assets/image%20%28173%29.png)

![](../.gitbook/assets/image%20%28127%29.png)

![](../.gitbook/assets/image%20%28239%29.png)



## 3. Improvement

### 3.1 框選問題1 - 放大鏡



![](../.gitbook/assets/image%20%28110%29.png)

![](../.gitbook/assets/image%20%28365%29.png)

![](../.gitbook/assets/image%20%28178%29.png)

![](../.gitbook/assets/image%20%28470%29.png)

![](../.gitbook/assets/image%20%28202%29.png)

![](../.gitbook/assets/image%20%2844%29.png)



```csharp
        /// <summary>
        /// 進入圖片區時，開啟放大功能(Timer Start)
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void pictureBox1_MouseEnter(object sender, EventArgs e)
        {
            if(pictureBox1.Image == null) { return; }
            timer1.Interval = 1 * 100;
            timer1.Start();
            
        }

        /// <summary>
        /// 離開圖片區時，取消放大功能(Timer Stop)
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void pictureBox1_MouseLeave(object sender, EventArgs e)
        {
            timer1.Stop();
            pictureBox2.Image = null;
        }

        /// <summary>
        /// 每0.1秒，顯示mouse目前區域且放大2倍示顯
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void timer1_Tick(object sender, EventArgs e)
        {
            try
            {
                int x = Control.MousePosition.X;
                int y = Control.MousePosition.Y;

                int magnification = 2;//倍率
                int imgWidth = pictureBox2.Width;
                int imgHeight = pictureBox2.Height;

                Bitmap bt = new Bitmap(imgWidth / magnification, imgHeight / magnification);
                using(Graphics g = Graphics.FromImage(bt))
                {
                    g.CopyFromScreen(
                         new Point(Cursor.Position.X - imgWidth / (2 * magnification),
                                   Cursor.Position.Y - imgHeight / (2 * magnification)),
                         new Point(0, 0),
                         new Size(imgWidth / magnification, imgHeight / magnification));
                    IntPtr dc1 = g.GetHdc();
                    g.ReleaseHdc(dc1);
                }

                pictureBox2.Image = (Image)bt;
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }
```

![](../.gitbook/assets/image%20%2894%29.png)

### 3.2 框選問題1 - Zoom In/Out

![](../.gitbook/assets/image%20%28325%29.png)

![](../.gitbook/assets/image%20%28312%29.png)



![](../.gitbook/assets/image%20%28209%29.png)

![](../.gitbook/assets/image%20%28305%29.png)

![](../.gitbook/assets/image%20%28366%29.png)

![](../.gitbook/assets/image%20%28137%29.png)

![](../.gitbook/assets/image%20%28335%29.png)

![](../.gitbook/assets/image%20%28182%29.png)

![](../.gitbook/assets/image%20%28338%29.png)

![](../.gitbook/assets/image%20%28165%29.png)

### 3.3 圖片處理 - 非同步

![](../.gitbook/assets/image%20%2810%29.png)

```csharp
private void btnGetTaggedImage_Click(object sender, EventArgs e)
        {
            try
            {
                if(dtLabels.Rows.Count == 0)
                {
                    return; //
                }
                string sImgPath = @".\img\";
                ImageHelper ih = new ImageHelper();
                string sLabelName = "";
                Bitmap source = new Bitmap(pictureBox1.Image);
                foreach(DataRow dr_label in dtLabels.Rows)
                {
                    sLabelName = dr_label["LABEL_NAME"].ToString();
                    Point p1 = new Point((int)dr_label["X1"], (int)dr_label["Y1"]);
                    Point p2 = new Point((int)dr_label["X2"], (int)dr_label["Y2"]);

                    //抓取標記的範圍
                    Bitmap img = ih.Crop(source, p1,p2);
                    img.Save($"{sImgPath}{sLabelName}-Step1-Ori.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //放大五倍
                    img = ih.Resize(img, img.Width * 5, img.Height * 5);
                    img.Save($"{sImgPath}{sLabelName}-Step2-Resize2.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //轉成灰階
                    img = ih.SetGrayscale(img);
                    img.Save($"{sImgPath}{sLabelName}-Step3-SetGrayscale.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //高斯模糊(為了解決文字解析度或字型造成缺口問題)
                    img = ih.GaussianBlur(img);
                    img.Save($"{sImgPath}{sLabelName}-Step4-GaussianBlur.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //轉成絕對黑白
                    img = ih.SetToBW(img, 190);
                    img.Save($"{sImgPath}{sLabelName}-Step5-ConvertTo1Bpp1.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //更新Image欄位
                    dr_label.BeginEdit();
                    dr_label["IMAGE"] = img;
                    dr_label["VAL"] = OCRHelper.OCR(img); //傳入圖片進行OCR
                    dr_label.EndEdit();
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }
```

```csharp
        private async void btnGetTaggedImage_Click(object sender, EventArgs e)
        {
            try
            {
                if(dtLabels.Rows.Count == 0)
                {
                    return; //
                }

                await ImagePreProcessAsync();

            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }
```

```csharp
        private async Task ImagePreProcessAsync()
        {
            Task[] tasks = new Task[dtLabels.Rows.Count];
            int iIndex = 0;

            foreach(DataRow dr_label in dtLabels.Rows)
            {
                //每一個Task處理Label (非同步方式執行)
                tasks[iIndex] = Task.Run(() => {
                    //處理圖片
                }
                );
            }

            Task.WaitAll(tasks); // 等待所有Task完成後
        }
```

```csharp
private async Task ImagePreProcessAsync()
        {
            Task[] tasks = new Task[dtLabels.Rows.Count];
            int iIndex = 0;

            string sImgPath = @".\img\";
            ImageHelper ih = new ImageHelper();
            //string sLabelName = "";

            foreach (DataRow dr_label in dtLabels.Rows)
            {
                Bitmap source = new Bitmap(pictureBox1.Image);
                
                //每一個Task處理Label (非同步方式執行)
                tasks[iIndex] = Task.Run(() => {

                    string sLabelName = dr_label["LABEL_NAME"].ToString();
                    Point p1 = new Point((int)dr_label["X1"], (int)dr_label["Y1"]);
                    Point p2 = new Point((int)dr_label["X2"], (int)dr_label["Y2"]);

                    //抓取標記的範圍
                    Bitmap img = ih.Crop(source, p1, p2);
                    img.Save($"{sImgPath}{sLabelName}-Step1-Ori.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //放大五倍
                    img = ih.Resize(img, img.Width * 5, img.Height * 5);
                    img.Save($"{sImgPath}{sLabelName}-Step2-Resize2.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //轉成灰階
                    img = ih.SetGrayscale(img);
                    img.Save($"{sImgPath}{sLabelName}-Step3-SetGrayscale.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //高斯模糊(為了解決文字解析度或字型造成缺口問題)
                    img = ih.GaussianBlur(img);
                    img.Save($"{sImgPath}{sLabelName}-Step4-GaussianBlur.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //轉成絕對黑白
                    img = ih.SetToBW(img, 190);
                    img.Save($"{sImgPath}{sLabelName}-Step5-ConvertTo1Bpp1.bmp", System.Drawing.Imaging.ImageFormat.Bmp);

                    //更新Image欄位
                    dr_label.BeginEdit();
                    dr_label["IMAGE"] = img;
                    dr_label["VAL"] = OCRHelper.OCR(img); //傳入圖片進行OCR
                    dr_label.EndEdit();
                }
                );
                iIndex++;
            }

            Task.WaitAll(tasks); // 等待所有Task完成後
        }
```

### 3.4 圖片處理 - 反底色問題

![](../.gitbook/assets/image%20%282%29.png)

![](../.gitbook/assets/image%20%28392%29.png)

![](../.gitbook/assets/image%20%2841%29.png)

```csharp
        public Bitmap Invert(Bitmap img)
        {
            Int32 th = 150;//150
            Int32 countBlack = 0;
            Int32 countWhite = 0;

            //轉成24bppRgb (for Set Pixel
            Bitmap bmp = img.Clone(new Rectangle(0, 0, img.Width, img.Height), System.Drawing.Imaging.PixelFormat.Format24bppRgb); 

            for (var x = 0; x < bmp.Width; x++)
            {
                for (var y = 0; y < bmp.Height; y++)
                {
                    var pixel = bmp.GetPixel(x, y);
                    if (pixel.R < th && pixel.G < th && pixel.B < th)
                    {
                        bmp.SetPixel(x, y, Color.Black);
                        countBlack++;
                    }
                    else
                    {
                        bmp.SetPixel(x, y, Color.White);
                        countWhite++;
                    }
                }
            }

            if (countBlack > countWhite)
            {
                //reverse
                for (var x = 0; x < bmp.Width; x++)
                {
                    for (var y = 0; y < bmp.Height; y++)
                    {
                        var pixel = bmp.GetPixel(x, y);
                        if (pixel.R < 220 && pixel.G < 220 && pixel.B < 220)
                        {
                            bmp.SetPixel(x, y, Color.White);
                        }
                        else
                        {
                            bmp.SetPixel(x, y, Color.Black);
                        }
                    }
                }
            }

            //轉回成8bppIndex
            Bitmap bmp2 = bmp.Clone(new Rectangle(0, 0, bmp.Width, bmp.Height), System.Drawing.Imaging.PixelFormat.Format8bppIndexed);

            return bmp2;
        }
```

![](../.gitbook/assets/image%20%2857%29.png)

![](../.gitbook/assets/image%20%28477%29.png)

![](../.gitbook/assets/image%20%28106%29.png)







