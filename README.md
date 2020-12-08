# Stock-Database
Manage and view records of your portfolio
from tkinter import *
from tkinter import messagebox
import requests
from bs4 import BeautifulSoup
import sqlite3
import matplotlib.pyplot as plt

root=Tk()
root.geometry("1100x400")
root.title("Stocks")
conn=sqlite3.connect("stock.db")
c=conn.cursor()

'''c.execute("""CREATE TABLE stocks (Symbol text,
				  Quantity real,
				  Purchase_Price real,
				  Price real,
				  Purchase_Total real,
				  Market_Value real,
				  Gain real,
				  Percent_Gain real)""")''' #Creates the database

def price_func(): #Finds the price of the ticker symbol entered for new database entries
    try:
        html_text = requests.get(f'https://finance.yahoo.com/quote/{symbol_e.get()}?p={symbol_e.get()}&.tsrc=fin-srch').text
        soup = BeautifulSoup(html_text, 'lxml')
        stk = soup.find('span', class_='Trsdu(0.3s) Fw(b) Fz(36px) Mb(-4px) D(ib)').text
        return(stk)
    except:
        response = messagebox.showerror("","Enter a valid ticker symbol")
def price_func2(tkr): #When the database is updated this pulls the ticker price in real time
    html_text = requests.get(f'https://finance.yahoo.com/quote/{tkr}?p={tkr}&.tsrc=fin-srch').text
    soup = BeautifulSoup(html_text, 'lxml')
    stk = soup.find('span', class_='Trsdu(0.3s) Fw(b) Fz(36px) Mb(-4px) D(ib)').text
    return(stk)
def submit(): #Updates the database
    conn=sqlite3.connect("stock.db")
    c=conn.cursor()
    ticker=symbol_e.get()
    ticker=ticker.upper()
    quantity=quantity_e.get()
    quantity=float(quantity)
    pprice=purchase_price_e.get()
    pprice=float(pprice)
    price=price_func()
    price=float(price)
    gain=((price-pprice)*quantity)
    mkt_value=price*quantity
    p_value=pprice*quantity
    pgain=((mkt_value-p_value)/p_value)*100
    
    c.execute("INSERT INTO stocks VALUES (:Symbol,:Quantity,:Purchase_Price,:Price,:Purchase_Total,:Market_Value,:Gain,:Percent_Gain)",
              {
                  'Symbol':ticker,
                  'Quantity':quantity,
                  'Purchase_Price':pprice,
                  'Price':price,
                  'Purchase_Total':(pprice*quantity),
                  'Market_Value':(price*quantity),
                  'Gain':gain,
                  'Percent_Gain':pgain,
                  })
    
    conn.commit()
    conn.close()
    symbol_e.delete(0,END)
    quantity_e.delete(0,END)
    purchase_price_e.delete(0,END)
def query(): #Displays the records in the database
    global query_label
    conn = sqlite3.connect("stock.db")
    c=conn.cursor()
    c.execute("SELECT *, oid FROM stocks")
    records=c.fetchall()

    print_records=''
    for record in records:
        ticker = record[0]
        ticker=ticker.upper()
        quantity = record[1]
        quantity=float(quantity)
        pprice = record[2]
        pprice=float(pprice)
        price=price_func2(record[0])
        price=float(price)
        gain=((price-pprice)*quantity)
        mkt_value=price*quantity
        p_value=pprice*quantity
        pgain=((mkt_value-p_value)/p_value)*100
        c.execute('''UPDATE stocks SET
                    Symbol = :ticker_,
                    Quantity = :quantity_,
                    Purchase_Price = :pprice_,
                    Price = :price_,
                    Purchase_Total = :p_value_,
                    Market_Value = :mkt_value_,
                    Gain = :gain_,
                    Percent_Gain = :pgain_

                    Where oid = :record''',
                      
                  {'ticker_': ticker,
                   'quantity_': quantity,
                   'pprice_': pprice,
                   'price_': price,
                   'p_value_': p_value,
                   'mkt_value_': mkt_value,
                   'gain_': gain,
                   'pgain_': pgain,
                   'record': record[8]})
        print_records += f'Record: {record[8]} | Ticker: {record[0]} | Qty: {record[1]} | Purchase price: ${record[2]} | Price: ${record[3]} | Purchase total: ${record[4]} | Mkt value: ${record[5]} | $ gain: ${record[6]} | % gain: {record[7]}% \n'

    query_label.grid_forget()
    query_label=Label(root,text=print_records)
    query_label.grid(row=5,column=0,columnspan=2)
    
    conn.commit()
    conn.close()
def delete_pop(): #Opens the delete database number window
    top=Toplevel()
    top.title("Delete")
    global delete_box
    delete_box_label=Label(top, text="Stock Ticker")
    delete_box_label.grid(row=0,column=0)
    delete_box=Entry(top, width=30)
    delete_box.grid(row=1,column=0)
    btn=Button(top, text="Close Window", command=top.destroy)
    btn.grid(row=2,column=1)
    btn2=Button(top, text="Delete Record #", command=delete)
    btn2.grid(row=2,column=0)
def delete():
    global delete_box
    conn = sqlite3.connect("stock.db")
    c=conn.cursor()

    c.execute("DELETE from stocks WHERE OID=" + delete_box.get())
    delete_box.delete(0,END)
    
    conn.commit()
    conn.close()
def update_pop():
    try:
        global pop
        global update_purchase_e
        global update_price_e
        global update_e1

        pop=Tk()
        pop.title("Update")
        conn=sqlite3.connect("stock.db")
        c=conn.cursor()
        
        record_id=update_e.get()
        c.execute("SELECT * FROM stocks WHERE oid=" + record_id)
        records=c.fetchall()
         
        update_box_label=Label(pop, text="Stock Ticker")
        update_box_label.grid(row=0,column=0)
        update_e1=Entry(pop)
        update_e1.grid(row=0,column=1)
        update_purchase_label=Label(pop, text='# Bought / Sold')
        update_purchase_label.grid(row=1,column=0)
        update_purchase_e=Entry(pop)
        update_purchase_e.grid(row=1,column=1)
        update_price_label=Label(pop, text='Price Bought / Sold')
        update_price_label.grid(row=2,column=0)
        update_price_e=Entry(pop)
        update_price_e.grid(row=2,column=1)
        btn_buy=Button(pop, text="Buy", command=buy)
        btn_buy.grid(row=3,column=0)
        btn_sell=Button(pop, text="Sell", command=sell)
        btn_sell.grid(row=3,column=1)

        for record in records:
            update_e1.insert(0, record[0])

        conn.commit()
        conn.close()
    except:
        response = messagebox.showerror("","Enter a record #")
def sell():
    try:
        conn = sqlite3.connect("stock.db")
        c=conn.cursor()
        c.execute("SELECT * FROM stocks WHERE oid=" + update_e.get()) #Selects everything from the ticker symbol in the update_pop function
        records=c.fetchall() #sets records to the list selected above | records[0][0]==JPM etc.
        quantity=(float(records[0][1])-float(update_purchase_e.get()))
        pprice=float(records[0][2])
        price=price_func2(records[0][0])
        price=float(price)
        p_value=pprice*quantity
        mkt_value=price*quantity
        gain=mkt_value-p_value
        pgain=((mkt_value-p_value)/p_value)*100

        c.execute('''UPDATE stocks SET
            Quantity = :quantity_,
            Price = :price_,
            Purchase_Total = :p_value,
            Market_Value = :mkt_value_,
            Gain = :gain_,
            Percent_Gain = :pgain_

            Where Symbol = :tck_''',

           {'quantity_': quantity,
            'price_': price,
            'p_value': p_value,
            'mkt_value_': mkt_value,
            'gain_': gain,
            'pgain_': pgain,
            'tck_': records[0][0]})
        
        conn.commit()
        conn.close()
        
        pop.destroy()
    except:
        response = messagebox.showerror("","Please fill out form")
def buy():
    try:
        conn = sqlite3.connect("stock.db")
        c=conn.cursor()
        c.execute("SELECT * FROM stocks WHERE oid=" + update_e.get()) #Selects everything from the ticker symbol in the update_pop function
        records=c.fetchall() #sets records to the list selected above | records[0][0]==JPM etc.
        quantity=(float(records[0][1])+float(update_purchase_e.get())) #[1]=quantity [2]=price [3]=
        price=price_func2(records[0][0])
        price=float(price)
        p_value=(float(records[0][4])+(float(update_purchase_e.get())*float(update_price_e.get())))
        pprice=p_value/quantity
        mkt_value=price*quantity
        gain=mkt_value-p_value
        pgain=((mkt_value-p_value)/p_value)*100

        c.execute('''UPDATE stocks SET
            Quantity = :quantity_,
            Purchase_Price = :pprice_,
            Price = :price_,
            Purchase_Total = :p_value,
            Market_Value = :mkt_value_,
            Gain = :gain_,
            Percent_Gain = :pgain_

            Where Symbol = :tck_''',

           {'quantity_': quantity,
            'pprice_': pprice,
            'price_': price,
            'p_value': p_value,
            'mkt_value_': mkt_value,
            'gain_': gain,
            'pgain_': pgain,
            'tck_': records[0][0]})
        
        conn.commit()
        conn.close()
        
        pop.destroy()
    except:
        response = messagebox.showerror("","Please fill out form")
def pie():
    conn=sqlite3.connect("stock.db")
    c=conn.cursor()

    c.execute("SELECT *, oid FROM stocks")
    records=c.fetchall()
    size=[]
    label=[]
    explode=[]
    for record in records:
        label.append(record[0])
        size.append(record[5])
        explode.append(.02)

    plt.pie(size,explode=explode,labels=label,autopct='%1.2f%%')
    plt.axis('equal')
    plt.show()
    
    conn.commit()
    conn.close()
def gui(): #Creates the main window
    global symbol_e
    global quantity_e
    global purchase_price_e
    global submit_b
    global query_btn
    global query_label
    global delete_btn
    global update_e
    global update_btn

    symbol_l=Label(root,text="Stock Ticker")
    symbol_l.grid(row=0,column=0)
    symbol_e=Entry(root,width=30)
    symbol_e.grid(row=0,column=1)

    quantity_l=Label(root,text="Add Quantity")
    quantity_l.grid(row=1,column=0)
    quantity_e=Entry(root,width=30)
    quantity_e.grid(row=1,column=1)

    purchase_price_l=Label(root,text="Purchase Price")
    purchase_price_l.grid(row=2,column=0)
    purchase_price_e=Entry(root,width=30)
    purchase_price_e.grid(row=2,column=1)

    submit_b=Button(root,text="Submit",command=submit)
    submit_b.grid(row=3,column=0,columnspan=2)

    query_btn=Button(root,text="Show Records", command=query)
    query_btn.grid(row =4,column=0,columnspan=2)

    query_label=Label(root,text='')
    query_label.grid(row=5,column=0,columnspan=2)

    delete_btn=Button(root,text="Delete Record", command=delete_pop)
    delete_btn.grid(row=6,column=0,columnspan=2)

    update_e=Entry(root)
    update_e.grid(row=7,column=0,columnspan=2)

    update_btn=Button(root,text="Update Record #", command=update_pop)
    update_btn.grid(row=8,column=0,columnspan=2)

    pie_btn=Button(root, text="PIE!", command=pie)
    pie_btn.grid(row=9, column=0, columnspan=2)

gui()

conn.commit()
conn.close()
root.mainloop()
