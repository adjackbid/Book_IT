# Tesseract OCR

## 1. Concept / Request

需求：老舊設備中機台電腦上，操作程式中的參數取得

限制：該程式無法將相關參數拋送

![](../.gitbook/assets/image%20%28109%29.png)

作法：在此限制下，執行概念如下

![](../.gitbook/assets/image%20%28333%29.png)

## 2. Develop

方便示意，假設機台程式UI如下，需要抓取的值為紅框所示

![](../.gitbook/assets/image%20%2849%29.png)

To Do List：

* [ ] 提供設定圖片要抓取位置的功能\(坐標、名稱\) =&gt; Labeling
* [ ] 將每一個設定好的位置之影像截取出來並進行一定程度處理 =&gt; Preprocess
* [ ] 針對每一個處理過的圖片進行OCR辨示圖片中的文字

Visual Studio C\#為例進行程式開發

### 2-1. Labeling

Labeling部分需要的功能有：

* [ ] 讀取圖片並顯示在操作界面中
* [ ] 框選範圍並可以針對該範圍給予一個變數名稱
* [ ] 儲存所有選取設定值

Visual Studio \(2017/2019\)建立新專案

選擇Windows Forms APP \(.Net Framework\)

![](../.gitbook/assets/image%20%28102%29.png)

![](../.gitbook/assets/image%20%28290%29.png)



![](../.gitbook/assets/image%20%2825%29.png)

![](../.gitbook/assets/image%20%28219%29.png)

![](../.gitbook/assets/image%20%28159%29.png)

![](../.gitbook/assets/image%20%28341%29.png)

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

![](../.gitbook/assets/image%20%28230%29.png)

![](../.gitbook/assets/image%20%28128%29.png)



![](../.gitbook/assets/image%20%2842%29.png)

![](../.gitbook/assets/image%20%28158%29.png)

![](../.gitbook/assets/image%20%28279%29.png)

```csharp
private void pictureBox1_MouseDown(object sender, MouseEventArgs e)
        {
            
            if (pictureBox1.Image == null) { return; }
            isMouseDown = true;
            pLeftUpper = e.Location;
            Refresh();
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

![](../.gitbook/assets/image%20%28223%29.png)

### 2-2. Image preprocess

### 2-3. OCR\(Tesseract\)

## 3. Application



