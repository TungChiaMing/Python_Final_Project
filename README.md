# Python_Final_Project

#大數據與程式設計導論 - 期末專題  - 擷取104人力銀行職缺資料  - B10802137 董家鳴 四電子二乙
from bs4 import BeautifulSoup
from selenium import webdriver
import time
import pandas as pd

def sort(Mode):
    if (Mode == '1'):
        option = '符合度排序'
    elif(Mode == '2'):
        option = "日期排序"
    el = browser.find_element_by_id("js-sort")
    for option in el.find_elements_by_tag_name('option'):
        if option.text == option:
            option.click() # select() in earlier versions of webdriver
            break
def web_crawler_104(SearchMode):
    i=0
    last_time_list=[]  ##設一個list來存上一次js動態載入更新前資料，以在資料不超過100筆或日期都比設定時間大時，跳出迴圈
   
    while(1):
        print("擷取中···")
        html = browser.page_source
        soup = BeautifulSoup(html)
        company_list = soup.find("div", attrs={'id':"js-job-content"}).find_all('article')   ##constantly get the new data
        print("已擷取%d筆資料"%len(company_list))
        
            
        
        if(SearchMode == '1'):
            if(last_time_list == company_list ):  
                my_company_list = company_list
               
                break
            last_time_list = company_list 
            if(len(company_list)>=100):
                my_company_list = company_list
                
                break
            

        elif(SearchMode == '2'):

            Date_Lastest_str = company_list[-1].span.text.replace(' ','').replace('\n','')  ###get the oldest time of the new data
            Date_Lastest_struct = time.strptime ( Date_Lastest_str   ,   "%m/%d"  )
            if(last_time_list == company_list ):  
                my_company_list = company_list
                LessThanTimeBack_flag = 0
                
                break
            last_time_list = company_list 
            if (Date_Lastest_struct.tm_mon <= time_back_N_day.tm_mon  and  Date_Lastest_struct.tm_mday <= time_back_N_day.tm_mday):  
                j=0
                for j in range( 0,len(company_list) ):

                    if len(company_list[j].span.text.replace(' ','').replace('\n','')) != 0:  ## 104前面幾筆資料可能會沒有日期

                        company_time = time.strptime(  company_list[j].span.text.replace(' ','').replace('\n','')  ,     "%m/%d"  )

                        j=j+1
                        

                        if(company_time.tm_mon  <= time_back_N_day.tm_mon  and company_time.tm_mday <=time_back_N_day.tm_mday):
                            LessThanTimeBack_flag = 1
                            break
                break
        



        
        browser.execute_script("window.scrollTo( 0 ,  document.body.scrollHeight  )")   ## scroll the height of a page
        
        
        try:
            if soup.find_all(class_="js-more-page") !=None:    ### 從點第一個開始，會有好個more-page
                browser.find_elements_by_class_name ('js-more-page')[i].click()  ## 必須與更下面的more-page互動，所以弄成list，再選
                i=i+1
                #print('sucessful_click')
        except:   ##  因為一開始沒有可點選的按鈕，如果不加這個，會因為前面找不到按鈕程式就停下來
            pass 

        time.sleep(2) ## delay 一段時間等網頁回應再解析，否則會當掉

    if(SearchMode == '2' and LessThanTimeBack_flag):  ## 如果最後一筆資料日期比較大才進不來這個判斷式
        k=0
        company_list_TimeMatched = []  ##  重建一個符合時間的 company_list
        #print(j)  ##這邊有點奇怪，不知道為甚麼break 之後 j自動+1了
        for k in range(0,j-1):   #  j 是while迴圈break前符合需要時間的所有公司數量
            company_list_TimeMatched.append(company_list[k])
            k=k+1
        my_company_list = company_list_TimeMatched
        print("去掉不符合日期的資料，總共擷取%d筆資料"%len(my_company_list))
    return my_company_list    
def MyCompanyList(My_company_list):
    company_name_address=[]  ##因爬取到的string有包含公司名子和住址，且是以跳行做 字的分隔，所以我先放在一個list裡，之後再用split分開list的元素
    company_name=[]
    company_address=[]
    job_title=[]
    job_class=[]
    job_link=[]
    job_content=[]
    job_salary=[]
    i=-1

    for d in My_company_list:
        if(d.find("p" , class_="b-tit") ==None):  ##搜尋結果太少的話會出現小人，會讓後面程式list索引的時候出錯，因為它基本上沒資料            
            
            try:
                company_name_address.append(d.find_all('li')[1].a['title'])
                i=i+1

                company_name.append(company_name_address[i].split('\n')[0].replace('公司名：','')) 
                company_address.append(company_name_address[i].split('\n')[1].replace('公司住址：','').replace('公司地址：',''))
            except:
                company_name_address.append('無')
                i=i+1

                company_name.append('無') 
                company_address.append('無')
                '''  
                Why?company_name_address[i].split('\n')[0]:  因為company_name_address中含有很多個list，不能直接分割，分割後的第一個元素是名子
                Why?replace:  因為稍後會放入dict，所以不需要多餘的字
                ''' 
            try:
                job_title.append(d.a.text)  #職稱
            except:
                job_title.append('無')
            try:
                if("市"  in d.find_all('li')[2].text):  ##前幾筆li這個標籤底下是抓到縣市，而不是職務名稱，所以加判斷式
                    job_class.append("無")
                else:
                    job_class.append(d.find_all('li')[2].text)   ###職務名稱
            except:
                    job_class.append("無")
            
            try:
                job_link.append("https:"+ d.a['href'])  # 網址
            except:
                job_link.append('無')
            try:
                job_content.append(d.p.text)  # 工作內容
            except:
                job_content.append('無')
            try:
                job_salary.append(d.find('span' , attrs = {'class' , 'b-tag--default'}).text) # 薪資
            except:
                job_salary.append('無')
    try:    
        My_company_dict={"公司名稱":company_name,"地址":company_address,"職稱":job_title,"職務類別":job_class,"網址":job_link,"薪資":job_salary,"工作內容":job_content}
    
        df=pd.DataFrame(My_company_dict)
        df.to_excel("My_company_list.xlsx") 
        print("並在程式相同目錄下成功產生檔名為My_company_list的Excel檔案")#，最後總共%d筆資料" %len(My_company_list))
    except:
        print("發生未知的錯誤，可能是您的Excel沒有關閉，或是目前網頁有所瑕疵")
        try:
            print("在此輸出表格")
            print(My_company_dict)
        except:
            print("網頁有所瑕疵，請換其他搜尋的方式，再重新輸入網址")
            


##-----------------------------------------------------------Main---------------------------------------------------------
job_104_url = input("請輸入已搜尋過的104人力銀行網頁:")

try:
    browser = webdriver.Chrome()
    browser.get(job_104_url)
    search_mode = input("請輸入要查詢的模式: 1.符合度排序 2.日期排序  :")
    if (search_mode == '1'):
        sort(search_mode)

        web_crawler_104(search_mode)
        MyCompanyList(web_crawler_104(search_mode))


    elif(search_mode == '2'):

        sort(search_mode)

        time_back_N_day = input("請輸入要查詢的天數(若您網頁的資料龐大，建議輸入5天以內):")
        if(time_back_N_day.isnumeric()==False):
                print("請重新執行並輸入正確的數值!")
        else:
            time_back_N_day=int(time_back_N_day)
            within_N_day = time_back_N_day*24*60*60
            time_back_N_day = time.localtime(time.time()-within_N_day)


            web_crawler_104(search_mode)
            MyCompanyList(web_crawler_104(search_mode))
    else:
        print("請重新執行並輸入正確的選項!")
except:
    print("錯誤的網址，請重新執行")
