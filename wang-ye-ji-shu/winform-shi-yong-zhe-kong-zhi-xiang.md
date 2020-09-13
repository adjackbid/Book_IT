# Winform - 使用者控制項

![](../.gitbook/assets/image%20%28435%29.png)



![](../.gitbook/assets/image%20%28403%29.png)



![](../.gitbook/assets/image%20%28419%29.png)



![](../.gitbook/assets/image%20%28417%29.png)



![](../.gitbook/assets/image%20%28418%29.png)



![](../.gitbook/assets/image%20%28415%29.png)



![](../.gitbook/assets/image%20%28413%29.png)

![](../.gitbook/assets/image%20%28404%29.png)

![](../.gitbook/assets/image%20%28420%29.png)



![](../.gitbook/assets/image%20%28438%29.png)



![](../.gitbook/assets/image%20%28401%29.png)



![](../.gitbook/assets/image%20%28427%29.png)



![](../.gitbook/assets/image%20%28402%29.png)



![](../.gitbook/assets/image%20%28409%29.png)



![](../.gitbook/assets/image%20%28400%29.png)



![](../.gitbook/assets/image%20%28447%29.png)



![](../.gitbook/assets/image%20%28412%29.png)



![](../.gitbook/assets/image%20%28407%29.png)



![](../.gitbook/assets/image%20%28446%29.png)



![](../.gitbook/assets/image%20%28439%29.png)



![](../.gitbook/assets/image%20%28414%29.png)



![](../.gitbook/assets/image%20%28443%29.png)



![](../.gitbook/assets/image%20%28399%29.png)



```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Drawing;
using System.Data;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace CustomControl
{
    public partial class MyUserControl : UserControl
    {
        public MyUserControl()
        {
            InitializeComponent();
        }

        [Description("Button1預設值")]
        [Category("Custom Property")]
        public int iButton1_ClickCount { get; set; } = 0;

        [Description("Button1預設值")]
        [Category("Custom Property")]
        public int iButton2_ClickCount { get; set; } = 0;

        [Description("Button1文字")]
        [Category("Custom Property")]
        public string Button1_Text { 
            get { return button1.Text; } 
            set { button1.Text = value; }
        }

        [Description("Button2文字")]
        [Category("Custom Property")]
        public string Button2_Text
        {
            get { return button2.Text; }
            set { button2.Text = value; }
        }

        private void button1_Click(object sender, EventArgs e)
        {
            iButton1_ClickCount++;
            textBox1.Text = "button1 click";
            //Trigger Custom Event
            Button1_Fired();
        }

        private void button2_Click(object sender, EventArgs e)
        {
            iButton2_ClickCount++;
            textBox1.Text = "button2 click";
            //Trigger Custom Event
            Button2_Fired();
        }


        [Description("點選Button1事件")]
        [Category("Action")]
        public event EventHandler<MyButtonClickEventArgs> MyButton1Submitted;
        [Description("點選Button1事件")]
        [Category("Action")]
        public event EventHandler<MyButtonClickEventArgs> MyButton2Submitted;


        private void Button1_Fired()
        {
            if (MyButton1Submitted != null)
            {
                var args = new MyButtonClickEventArgs(iButton1_ClickCount);
                MyButton1Submitted(this, args);
            }
        }

        private void Button2_Fired()
        {
            if (MyButton2Submitted != null)
            {
                var args = new MyButtonClickEventArgs(iButton2_ClickCount);
                MyButton2Submitted(this, args);
            }
        }

    }
}

```

