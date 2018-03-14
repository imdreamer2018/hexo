title: ' 基于python3.6连接mysql，实现一个银行转账的小功能（源码）'
tags:
  - python
  - mysql
  - python-class
categories: []
date: 2018-02-02 16:34:00
---

基于python3.6连接mysql，实现一个银行转账的小功能（源码）
<!--more-->
{% codeblock lang:python%}
import pymysql


class TransferMoney:
    def __init__(self,conn):
        self.conn = conn
    def check_acct_available(self,acctid):
        try:
            cursor = self.conn.cursor()
            sql = "select * from account where acctid = %s" %acctid
            cursor.execute(sql)
            print("check_acct_available"+sql)
            rs = cursor.fetchall()
            if len(rs) !=1:
                raise Exception("account %s is not exist" % acctid)
        finally:
            cursor.close()
    def has_enough_money(self,acctid,money):
        try:
            cursor = self.conn.cursor()
            sql = "select * from account where acctid = %s and money>= %s" % (acctid,money)
            cursor.execute(sql)
            print("has_enough_money" + sql)
            rs = cursor.fetchall()
            if len(rs) != 1:
                raise Exception("account %s does't have enough money" % acctid)
        finally:
            cursor.close()
    def reduce_money(self,acctid,money):
        try:
            cursor = self.conn.cursor()
            sql = "update account set money = money - %s where acctid = %s" % (money,acctid)
            cursor.execute(sql)
            print("reduce_money" + sql)
            rs = cursor.rowcount
            if rs != 1:
                raise Exception("account %s transfer is't successful" % acctid)
        finally:
            cursor.close()
    def add_money(self,acctid,money):
        try:
            cursor = self.conn.cursor()
            sql = "update account set money = money + %s where acctid = %s" % (money,acctid)
            cursor.execute(sql)
            print("add_money" + sql)
            rs = cursor.rowcount
            if rs != 1:
                raise Exception("account %s transfer is't successful" % acctid)
        finally:
            cursor.close()
    def  transfer(self,source_acctid, target_acctid, money):
        try:
            self.check_acct_available(source_acctid)
            self.check_acct_available(target_acctid)
            self.has_enough_money(source_acctid,money)
            self.reduce_money(source_acctid,money)
            self.add_money(target_acctid,money)
            self.conn.commit()
        except Exception as e:
            self.conn.rollback()
            raise e
#if __name__ == "__main__":
    #source_acctid = sys.argv[1]
    #target_acctid = sys.argv[2]
    #money = sys.argv[3]

conn = pymysql.Connect(
    host = '*******',
    port =3306,
    user = 'root',
    passwd = '*************',
    db = 'python'

)
source_acctid = int(input("Input source_acctid"))
target_acctid = int(input("Input target_acctid"))
money = int(input("Input tansfer money "))
tr_money = TransferMoney(conn)
try:
    tr_money.transfer(source_acctid,target_acctid,money)
except Exception as e:
    print("raise Error"+str(e))
finally:
    conn.close()
	
{% endcodeblock %}