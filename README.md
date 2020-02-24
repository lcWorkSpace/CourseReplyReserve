# CourseReplyReserve
import os
import re
from datetime import datetime
from openpyxl import Workbook, load_workbook
from openpyxl.styles import Alignment, Font
from openpyxl.utils import get_column_letter
from prettytable import PrettyTable


def validate(date_text):
    """
    检测日期与日期格式是否正确
    :param date_text: 日期文本
    """
    try:
        if date_text != datetime.strptime(date_text, "%Y-%m-%d").strftime('%Y-%m-%d'):
            raise ValueError
        return date_text
    except ValueError:
        # raise ValueError("错误是日期格式或日期,格式是年-月-日")
        return False


def setCourse_time(course_time, student_num):
    """
    设置对应时间下的最大预约人数
    :param course_time: 教师有空的预约时间(str)
    :param student_num: 对应的最大预约人数(int)
    :return: 存储着各对应时间下的最大预约人数列表
    """
    List_student_num = []
    List_reCourseTime = course_time.split(" ")
    for time in range(1, 13):
        # 判断教师在哪个时间下有空，再设置相应的最大预约人数
        if str(time) in List_reCourseTime:
            List_student_num.append(int(student_num))
        else:
            List_student_num.append(0)
    return List_student_num


def setDate():
    """ 设置预约日期 """
    while True:
        date = input("请输入日期：（使用正确的格式，如：2020-03-01）\n")
        re_date = validate(date)
        if re_date:  # 日期符合格式则返回date，反之返回False
            return re_date
        else:
            print("日期或日期格式有误！请重新输入！")
            continue


def teacher_init_date():
    """ 教师信息初始化 """
    # 创建一个空列表，用于存储需写入excel中的日期信息
    List_date = []
    # 创建一个空列表，用于存储需写入excel中的时段信息
    List_time = []
    # 创建一个变量judge，用于控制程序
    judge = 1
    while True:
        if judge == 1:
            reDate = setDate()
            # 避免产生重复日期
            if len(List_date) < 1:
                List_date.append(reDate)
            else:
                if reDate == List_date[-1]:
                    print("{}日程安排有重复！".format(reDate))
                    continue
                else:
                    List_date.append(reDate)
            course_time = input("请输入时段：（时段按1-12节课划分，如1 2 4）\n")
            student_num = int(input("请输入每个时段可预约的人数：（每节课答辩人数少于5比较合适）\n"))
            List_maxReserveNum = setCourse_time(course_time, student_num)
            List_time.append(List_maxReserveNum)
        else:
            pass
        select = int(input("继续设置日期请输入1，预览预约设置请输入2，退出本角色请输入0\n"))
        if select == 1:
            judge = 1
            continue
        elif select == 2:
            judge = 2
            for i in range(len(List_time)):
                print("{}预约情况如下:".format(List_date[i]))
                # 利用列表生成式生成一个1到12的列表
                headers = PrettyTable(["第{}节".format(x) for x in range(1, 13)])
                # 列表生成式
                headers.add_row(["最大预约人数{}".format(item) for item in List_time[i]])
                print(headers)
        elif select == 0:
            return List_date, List_time
        else:
            judge = 2
            print("没有找到对应的功能，请重新选择！")
            continue


def write_to_excel(workbook, day, row, c_start, c_end, tuple_data):
    """
    将数据写入到excel中
    :param day: 预约天数
    :param workbook: 工作簿对象
    :param row: 所在行
    :param c_start: 所在行的起始列
    :param c_end: 所在行的终止列
    :param tuple_data: 元组形式的数据源
    """
    # 激活工作表单
    worksheet = workbook.active
    # 设置表单名称
    worksheet.title = "课程设计答辩预约情况"
    # 合并表单的单元格
    worksheet.merge_cells("A{0}:M{0}".format(row))
    # 设置合并单元格的文字字体与排版样式
    worksheet["A{}".format(row)].font = Font(sz=18, bold=True)
    worksheet["A{}".format(row)].alignment = Alignment(horizontal="center", vertical="center")
    # 将预约时间写入合并单元格中
    worksheet["A{}".format(row)] = tuple_data[0][day]
    # 创建两个列表，用于存储竖列表头数据与对应的相关数据
    List_sheetHeader = ["时段", "最大预约人数", "已预约人数", "\n已\n预\n约\n学\n生\n学\n号\n"]
    List_sheetData = ["第{}节", tuple_data[1][day], 0, "如下："]
    for r in range(row + 1, row + 5):
        # 设置竖列表头
        worksheet["A{}".format(r)] = List_sheetHeader[r - 2 - (day * 5)]
        # 将对应时间下的最大预约人数写入excel表中
        for col in range(c_start, c_end):
            data = List_sheetData[r - 2 - (day * 5)]
            if data == "第{}节":
                worksheet.cell(r, col, data.format(col - c_start + 1))
            elif type(data) is list:
                worksheet.cell(r, col, data[col - c_start])
            elif type(data) is int:
                worksheet.cell(r, col, data)
            else:
                worksheet["{0}{1}".format(get_column_letter(col), r)].alignment = \
                    Alignment(horizontal="center")
                worksheet.cell(r, col, data)


def show_excelData():
    """ 屏幕输出预约信息情况 """
    ws = load_workbook(r"reserve.xlsx")
    wb = ws["课程设计答辩预约情况"]
    for row in wb.rows:
        for cell in row:
            if cell.value or cell.value == 0:
                print(cell.value, end="\t")
            else:
                print(" ", end="\t")
        print("\n")


def create_excel(workbook):
    """
    生成excel表
    :param workbook: 工作薄对象
    """
    # 生成的excel表所在路径
    workbook.save(r"reserve.xlsx")


def addPerson_excel(s_num, row, col):
    """
    使已预约人数自增加value，然后保存修改
    :param s_num: 学生学号
    :param row: 数据自增加N的单元格所在行
    :param col: 数据自增加N的单元格所在列
    :return: 新的已预约人数
    """
    workbook = load_workbook(r"reserve.xlsx")
    worksheet = workbook["课程设计答辩预约情况"]
    cell_value = worksheet["{0}{1}".format(get_column_letter(col), row)].value
    cell_value += 1
    worksheet["{0}{1}".format(get_column_letter(col), row)] = cell_value
    student_num = worksheet["{0}{1}".format(get_column_letter(col), row + 1)].value + "\n" + s_num
    worksheet["{0}{1}".format(get_column_letter(col), row+1)] = student_num
    workbook.save(r"reserve.xlsx")
    return cell_value


def teacher_setReserve():
    """ 教师设置答辩预约表,并将信息存储到excel表格中 """
    # 用一个元组接收函数teacher_init_date()的返回值
    Tuple_teacherMsg = teacher_init_date()
    # 创建工作薄
    workbook = Workbook()
    # 创建新的变量new_pos与sheet_num，用于控制程序
    new_pos = 0
    day = 0
    for row in range(len(Tuple_teacherMsg[0])):
        # 调用write_to_excel()函数，将信息写入excel表
        write_to_excel(workbook, day, row + new_pos + 1, 2, 14, Tuple_teacherMsg)
        new_pos += 4
        day += 1
    create_excel(workbook)


def show_studentMsg(Dict_studentMsg):
    """ 显示学生预约信息 """
    d_key = ""
    d_value = ""
    for key, value in Dict_studentMsg.items():
        d_key = key
        d_value = value
    print("尊敬的{0}用户，您的预约信息如下：\n"
          "{1}".format(d_key, d_value))


def student_appointJudge(s_num, c_start, c_end):
    """ 学生预约判定 """
    if os.path.exists(r"reserve.xlsx"):
        wb = load_workbook(r"reserve.xlsx")
        ws = wb["课程设计答辩预约情况"]
        # 创建空字典存储学生信息
        Dict_studentData = {}
        Dict_studentMsg = {s_num: Dict_studentData}
        # 查找列表中是否有对应学号的学生信息
        for row in range(5, ws.max_row + 1, 5):
            for col in range(c_start, c_end):
                if ws["{0}{1}".format(get_column_letter(col), row)].value.find(s_num) == 1:
                    print(ws["{0}{1}".format(get_column_letter(col), row)].value)
                    Dict_studentData["日期"] = ws["{0}{1}".format(get_column_letter(1), row - 4)].value
                    Dict_studentData["时段"] = ws["{0}{1}".format(get_column_letter(col), row - 3)].value
                    Dict_studentData["序号"] = ws["{0}{1}".format(get_column_letter(col), row - 1)].value
                    return Dict_studentMsg
        return 1
    else:
        return False


def road_excel_data(c_start, c_end):
    """ 读取excel中的数据, 返回数据字典 """
    Dict_data = {}
    List_all_data = []
    List_date = []
    wb = load_workbook(r"reserve.xlsx")
    ws = wb["课程设计答辩预约情况"]
    for row in range(1, ws.max_row + 1, 5):
        List_date.append(ws["A{}".format(row)].value)
        List_time = []
        List_now_person = []
        List_max_person = []
        Dict_msg = {"时段": List_time, "每时段最大预约人数": List_max_person, "已预约人数": List_now_person}
        for col in range(c_start, c_end):
            # if ws["{0}{1}".format(get_column_letter(col), row + 2)] != 0:
            List_time.append(ws["{0}{1}".format(get_column_letter(col), row + 1)].value)
            List_max_person.append(ws["{0}{1}".format(get_column_letter(col), row + 2)].value)
            List_now_person.append(ws["{0}{1}".format(get_column_letter(col), row + 3)].value)
            # Dict_msg = {"时段": List_time, "每时段最大预约人数": List_max_person, "已预约人数": List_now_person}
        List_all_data.append(Dict_msg)
    # print(List_all_data)
    for d in range(len(List_date)):
        Dict_data[List_date[d]] = List_all_data[d]
    return Dict_data


def student_judgeData(reserveDate, reserveTime):
    List_date = []
    row = 0
    Dict_data = road_excel_data(2, 14)
    # print(Dict_data)
    for key in Dict_data.keys():
        List_date.append(key)
    # print(List_date)
    if reserveDate in List_date:
        for i in range(len(List_date)):
            if reserveDate == List_date[i]:
                row = i + 4
        if "第{}节".format(reserveTime) in Dict_data[reserveDate]["时段"]:
            col = int(reserveTime) - 1
            if Dict_data[reserveDate]["每时段最大预约人数"][col] > 0:
                if Dict_data[reserveDate]["已预约人数"][col] < \
                        Dict_data[reserveDate]["每时段最大预约人数"][col]:
                    Tuple_backValue = (row, col+2)
                else:
                    Tuple_backValue = (-4,)
            else:
                Tuple_backValue = (-3,)
        else:
            Tuple_backValue = (-2,)
    else:
        Tuple_backValue = (-1,)
    return Tuple_backValue


def student_reserve(s_num):
    """ 学生进行预约 """
    while True:
        reserveDate = input("请选择预约日期：（使用正确的格式，如：2020-03-01）\n")
        re_reserveDate = validate(reserveDate)
        reserveTime = int(input("请选择预约时段：（时段按1-12节课划分）\n"))
        Tuple_result = student_judgeData(re_reserveDate, reserveTime)
        if Tuple_result[0] == -1:
            print("{}暂无老师可预约进行答辩，请重新选择！".format(re_reserveDate))
            continue
        elif Tuple_result[0] == -2:
            print("{0}不存在第{1}节时段，请重新选择！".format(re_reserveDate, reserveTime))
            continue
        elif Tuple_result[0] == -3:
            print("{0}第{1}节时段暂无老师可预约进行答辩，请重新选择！".format(re_reserveDate, reserveTime))
            continue
        elif Tuple_result[0] == -4:
            print("{0}第{1}节时段预约人数已满，请重新选择！".format(re_reserveDate, reserveTime))
            continue
        else:
            # 创建变量select用于控制while循环
            select = int(input("确定请输入1，重选请输入2，退出本角色请输入0\n"))
            if select == 1:
                new_reservePerson = addPerson_excel(s_num, Tuple_result[0], Tuple_result[1])
                print("您的预约信息为{0}第{1}节时段第{2}个，请谨记！"
                      .format(re_reserveDate, reserveTime, new_reservePerson))
                break
            elif select == 2:
                continue
            elif select == 0:
                break
            else:
                print("没有找到对应的功能，请重新输入数据！")
                continue


if __name__ == '__main__':
    __pattern = re.compile(r"\d{12}")
    print(" " * 4, "欢迎使用课程设计答辩预约系统")
    while True:
        choice = int(input("教师请输入1，学生请输入2, 输入0退出程序\n"))
        label = input("请输入正确的用户标识:(退出程序时可按回车键跳过)\n")
        if choice == 1 and label == "teacher":
            t_func_opt = int(input("————func menu————\n"
                                   "1.设置预约答辩日期和时段\n"
                                   "2.查看总体预约情况\n"
                                   "请输入需执行功能前的序号...\n"))
            if t_func_opt == 1:
                teacher_setReserve()
                continue
            elif t_func_opt == 2:
                if os.path.exists(r"reserve.xlsx"):
                    show_excelData()
                else:
                    print("您还未设置预约答辩日期和时段，请先进行设置！")
                continue
        elif choice == 2 and __pattern.match(label):
            # 定义一个变量rest保存函数student_appointJudge()的返回值
            rest = student_appointJudge(label, 2, 14)
            s_func_opt = int(input("————func menu————\n"
                                   "1.预约答辩日期和时段\n"
                                   "2.查看自己的预约情况\n"
                                   "请输入需执行功能前的序号...\n"))
            if (type(rest) is dict) and s_func_opt == 2:
                # 输出对应学号的学生预约信息
                show_studentMsg(rest)
                continue
            elif (type(rest) is int) and s_func_opt == 1:
                print("目前课程设计答辩预约情况如下：\n")
                show_excelData()
                student_reserve(label)
                continue
            elif (type(rest) is int) and s_func_opt == 2:
                print("您还未预约答辩日期和时段，请先预约！")
                continue
            elif (type(rest) is dict) and s_func_opt == 1:
                print("您已预约了答辩日期和时段，请勿重复预约！")
                continue
            elif type(rest) is bool:
                print("系统还未有老师设置的答辩日期和时段，请稍后再试！")
                continue
            else:
                print("输入有误，请重新输入！")
                continue
        elif choice == 0:
            break
        else:
            print("输入有误，请重新输入！")
            continue
