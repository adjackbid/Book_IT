# Crystal Report - Label Printing

![](../.gitbook/assets/image%20%28148%29.png)

Crystal Report一般使用在報表用途，可以自定訂欄位及樣式，在給定資料來源可以產生報表並提供匯出功能。

此特性，也可以被應用在列印標籤、發票程式中，若沒有使用標籤軟體時，使用免費版的Crystal Report進行開發會是個不錯的選擇

## Prepare

下載Crystal Report Runtime /IDE for .Net Framework

{% embed url="https://www.crystalreports.com/crvs/confirm/" %}

![](../.gitbook/assets/image%20%28291%29.png)

## Demo

建立Demo專案

![](../.gitbook/assets/image%20%28263%29.png)

![](../.gitbook/assets/image%20%2883%29.png)

新增四個TextBox及一個列印Button

![](../.gitbook/assets/image%20%28225%29.png)

Print Click事件 - 取得輸入的參數並呼叫PrintLabel Function\(暫不寫code\)

![](../.gitbook/assets/image%20%28315%29.png)

加入資料集\(DataSet\) - 做為Crystal Report的資料來源

![](../.gitbook/assets/image%20%28267%29.png)

![](../.gitbook/assets/image%20%2887%29.png)

DataSet建立完成如下，可從左方工具箱新增DataTable至設計介面中

![](../.gitbook/assets/image%20%2886%29.png)

![](../.gitbook/assets/image%20%28284%29.png)

在DataTable中新增資料行\(Column\)

![](../.gitbook/assets/image%20%28309%29.png)

此範例，需要四個欄位如下，可視情況調整欄位的DataType \(string / int...\)

![](../.gitbook/assets/image%20%28216%29.png)

![](../.gitbook/assets/image%20%28243%29.png)

新增Crystal Report物件

![](../.gitbook/assets/image%20%2810%29.png)

選擇使用空白報表

![](../.gitbook/assets/image%20%28254%29.png)

在欄位總管中 - 點選資料庫欄位 →右鍵點選資料庫專家

![](../.gitbook/assets/image%20%28215%29.png)

點選資料案資料→ADO.NET資料集→可以找到自行建立的DataSet - DSLabel中的DataTable1

![](../.gitbook/assets/image%20%28177%29.png)

將DataTable1加入至右方

![](../.gitbook/assets/image%20%28100%29.png)

確認後，資料庫欄位則會出現DataTable1及對應的欄位可以使用

![](../.gitbook/assets/image%20%28151%29.png)

接著可以將欄位直接拉至畫面中

![](../.gitbook/assets/image%20%28112%29.png)

一次會建立兩個物件：

Text Object：預設值會為欄位名稱\(可修改\)，若不需要可以砍掉

Field Object：該欄位在資料中對應到的值

![](../.gitbook/assets/image%20%28289%29.png)

將FROM的Text Object調整為「FROM：」並調整字型

![](../.gitbook/assets/image%20%28409%29.png)

調整位置

![](../.gitbook/assets/image%20%28383%29.png)

插入線條

![](../.gitbook/assets/image%20%2830%29.png)

![](../.gitbook/assets/image%20%28357%29.png)

依此類推，拉出Label的樣式

![](../.gitbook/assets/image%20%28341%29.png)

其中ItemNo區，下方有一個BarCode條碼，因此再建一個ItemNo物件，將欄位名稱刪除

![](../.gitbook/assets/image%20%28369%29.png)

將Field Object的字型調整成條碼字型

![](../.gitbook/assets/image%20%28178%29.png)

完成如下 → 存檔

![](../.gitbook/assets/image%20%2877%29.png)

修改PrintLabel方法

```csharp
    private void PrintLabel(string sFrom, string sTo, string sWeight, string sItemNo)
        {
            DSLabel ds = new DSLabel();

            //CREATE DataTable
            DataTable dt = ds.Tables[0];
            DataRow dr_row = null;
            dr_row = dt.NewRow();
            dr_row["FROM"] = sFrom;
            dr_row["TO"] = sTo;
            dr_row["WEIGHT"] = sWeight;
            dr_row["ITEMNO"] = sItemNo;
            dt.Rows.Add(dr_row);

            //Create CR Report Object
            ReportDocument doc = new CRLabel();
            doc.SetDataSource(ds);

            //Print
            doc.PrintToPrinter(1, false, 0, 0);
        }
```

測試

![](../.gitbook/assets/image%20%28159%29.png)

列印結果

![](../.gitbook/assets/image%20%28408%29.png)

完整Sample Code

```csharp
using CrystalDecisions.CrystalReports.Engine;
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace CrystalReportDemo
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();
        }

        private void btnPrint_Click(object sender, EventArgs e)
        {
            try
            {
                string sFrom = ttbFrom.Text.Trim();
                string sTo = ttbTo.Text.Trim();
                string sWeight = ttbWeight.Text.Trim();
                string sItemNo = ttbItemNo.Text.Trim();

                PrintLabel(sFrom, sTo, sWeight, sItemNo);

            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }

        private void PrintLabel(string sFrom, string sTo, string sWeight, string sItemNo)
        {
            DSLabel ds = new DSLabel();

            //CREATE DataTable
            DataTable dt = ds.Tables[0];
            DataRow dr_row = null;
            dr_row = dt.NewRow();
            dr_row["FROM"] = sFrom;
            dr_row["TO"] = sTo;
            dr_row["WEIGHT"] = sWeight;
            dr_row["ITEMNO"] = sItemNo;
            dt.Rows.Add(dr_row);

            //Create CR Report Object
            ReportDocument doc = new CRLabel();
            doc.SetDataSource(ds);

            //Print
            doc.PrintToPrinter(1, false, 0, 0);
        }
    }
}

```

## Source Code

Demo用完整Source Code：

[https://github.com/adjackbid/CrystalReportDemo](https://github.com/adjackbid/CrystalReportDemo)

