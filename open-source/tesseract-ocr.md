# Tesseract OCR

Tesseract OCR是一個開源OCR Opensource，可針對各國語言進行OCR

若條件算不複雜的需求時，可以使用這個套件解進OCR問題

{% embed url="https://github.com/tesseract-ocr/tesseract" %}

## 1. Concept / Request

需求：老舊設備中機台電腦上，操作程式中的參數取得

限制：該程式無法將相關參數拋送

![](../.gitbook/assets/image%20%28123%29.png)

作法：在此限制下，執行概念如下

![](../.gitbook/assets/image%20%28389%29.png)

## 2. Develop

方便示意，假設機台程式UI如下，需要抓取的值為紅框所示

![](../.gitbook/assets/image%20%2854%29.png)

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

![](../.gitbook/assets/image%20%28116%29.png)

![](../.gitbook/assets/image%20%28339%29.png)

畫面左方中加入一個Button - Load Image：為了選取圖片來源

![](../.gitbook/assets/image%20%2829%29.png)

畫面右方新增一個pictureBox：顯示圖片用，因為圖片大小可能不會等於PictureBox大小，所以Size Mode要選擇Zoom，讓程式自動縮放圖片

![](../.gitbook/assets/image%20%28255%29.png)

![](../.gitbook/assets/image%20%28183%29.png)

Button LoadImage Click事件處理

![](../.gitbook/assets/image%20%28398%29.png)

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

![](../.gitbook/assets/image%20%28270%29.png)

選取檔案後，顯示在右方pictureBox中





![](../.gitbook/assets/image%20%28148%29.png)

接著為了要做到讓使用者框選一個區塊的功能\(例如小畫家中的框選功能\)

![](../.gitbook/assets/image%20%2847%29.png)

針對PictureBox1中的Mouse Down / Up / Move事件進行處理，並新增幾個變數紀錄需要的座標\(開始座標\(左上XY\)、結束座標\(右下XY\)\)

![](../.gitbook/assets/image%20%28182%29.png)

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

![](../.gitbook/assets/image%20%2862%29.png)

## 

測試OK，可以依照選取的位置進行框選動作

![](../.gitbook/assets/image%20%28284%29.png)

為方便顯示座標位置，新增一個richtextbox

![](../.gitbook/assets/image%20%28131%29.png)

加入方法回報座標\(傳入point\)並在mouse down / up / move事件中呼叫回報方法\(ReportLocation\)

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

測試可以正常顯示座標位置

![](../.gitbook/assets/image%20%28382%29.png)

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

![](../.gitbook/assets/image%20%28214%29.png)

因為PictureBox是整個右半邊，因此X,Y起始點會在藍色標記位置，但圖片因為等比例縮放關係，不會對齊左上角，圖片的X、Y起始位置應為綠色框選位置

![](../.gitbook/assets/image%20%28110%29.png)

![](../.gitbook/assets/image%20%2877%29.png)

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

![](../.gitbook/assets/image%20%28319%29.png)

解決座標問題後，接著新增DataGridView，以顯示、記錄每一個框選的Label名稱、座標資料

![](../.gitbook/assets/image%20%28277%29.png)

新增DataTable dtLabels並指定GridView的DataSource為dtLabels

![](../.gitbook/assets/image%20%28240%29.png)



![](../.gitbook/assets/image%20%28125%29.png)

![](../.gitbook/assets/image%20%28293%29.png)



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

![](../.gitbook/assets/image%20%28248%29.png)

### 2-2. Image preprocess

* [ ] 讀取Label設定 - 取得座標位置並將對應圖片取出
* [ ] 圖片處理

![](../.gitbook/assets/image%20%28306%29.png)

![](../.gitbook/assets/image%20%28395%29.png)

![](../.gitbook/assets/image%20%28341%29.png)

### 2-3. OCR\(Tesseract\)

## 3. Application



