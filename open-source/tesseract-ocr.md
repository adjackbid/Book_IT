# Tesseract OCR

## 1. Concept / Request

需求：老舊設備中機台電腦上，操作程式中的參數取得

限制：該程式無法將相關參數拋送

![](../.gitbook/assets/image%20%28113%29.png)

作法：在此限制下，執行概念如下

![](../.gitbook/assets/image%20%28345%29.png)

## 2. Develop

方便示意，假設機台程式UI如下，需要抓取的值為紅框所示

![](../.gitbook/assets/image%20%2850%29.png)

To Do List：

* [x] 提供設定圖片要抓取位置的功能\(坐標、名稱\) =&gt; Labeling
* [x] 將每一個設定好的位置之影像截取出來並進行一定程度處理 =&gt; Preprocess
* [x] 針對每一個處理過的圖片進行OCR辨示圖片中的文字

Visual Studio C\#為例進行程式開發

### 2-1. Labeling

Labeling部分需要的功能有：

* [x] 讀取圖片並顯示在操作界面中
* [x] 框選範圍並可以針對該範圍給予一個變數名稱
* [x] 儲存所有選取設定值

Visual Studio \(2017/2019\)建立新專案

選擇Windows Forms APP \(.Net Framework\)

![](../.gitbook/assets/image%20%28106%29.png)

![](../.gitbook/assets/image%20%28300%29.png)



![](../.gitbook/assets/image%20%2825%29.png)

![](../.gitbook/assets/image%20%28227%29.png)

![](../.gitbook/assets/image%20%28164%29.png)

![](../.gitbook/assets/image%20%28353%29.png)

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

![](../.gitbook/assets/image%20%28238%29.png)

![](../.gitbook/assets/image%20%28133%29.png)



![](../.gitbook/assets/image%20%2843%29.png)

![](../.gitbook/assets/image%20%28163%29.png)

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

![](../.gitbook/assets/image%20%2858%29.png)

![](../.gitbook/assets/image%20%28250%29.png)



![](../.gitbook/assets/image%20%28120%29.png)

```csharp
        private void ReportLoaction(Point location1)
        {
            richTextBox1.Text = $"X={location1.X}\r\nY={location1.Y}";
        }
```

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

![](../.gitbook/assets/image%20%28338%29.png)



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

![](../.gitbook/assets/image%20%28192%29.png)

![](../.gitbook/assets/image%20%28100%29.png)

![](../.gitbook/assets/image%20%2869%29.png)

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

![](../.gitbook/assets/image%20%28281%29.png)

![](../.gitbook/assets/image%20%2835%29.png)

![](../.gitbook/assets/image%20%28216%29.png)





### 2-2. Image preprocess

### 2-3. OCR\(Tesseract\)

## 3. Application



