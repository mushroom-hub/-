# -学生信息管理系统简介
【系统概述】
这是一个基于Python和Tkinter开发的桌面应用程序，用于管理学生基本信息。系统采用SQLite数据库存储数据，提供了直观的图形用户界面(GUI)，实现了学生信息的增删改查(CRUD)基本功能。

【主要功能】
1.学生信息管理：
添加新学生信息（姓名、学号、性别、年龄、专业、邮箱）
编辑现有学生信息
删除学生记录
查看所有学生列表
2.搜索功能：
支持按姓名、学号或专业模糊搜索
快速定位特定学生信息
3.数据展示：
表格形式清晰展示所有学生信息
支持滚动浏览大量数据
列宽可调整，信息展示完整

【技术特点】
1.前端技术：
使用Python标准库Tkinter构建GUI界面
采用Treeview控件高效展示表格数据
合理的布局和控件设计，操作直观
2.后端技术：
SQLite轻量级数据库存储数据
参数化查询防止SQL注入
完善的错误处理和用户反馈
3.交互设计：
支持键盘快捷键（如回车键搜索）
双击快速编辑
删除前确认提示防止误操作

【适用场景】
1.学校教务管理
2.培训班学员管理
3.小型教育机构学生信息维护

【优势】
1.轻量便携：单文件数据库，无需复杂安装
2.简单易用：界面直观，操作简单明了
3.安全可靠：数据验证和错误处理完善
4.跨平台：可在Windows、macOS、Linux等系统运行

这个系统虽然功能简洁，但涵盖了完整的信息管理系统所需的基本要素，适合作为学习项目或小型机构使用。



    import tkinter as tk
    from tkinter import ttk, messagebox, Menu
    import sqlite3

    class StudentManagementSystem:
      def __init__(self, root):
        self.root = root
        self.root.title("学生信息管理系统")
        self.root.geometry("900x600")
        # 创建数据库和表
        self.create_database()
        # 创建GUI界面
        self.create_menu()
        self.create_search_frame()
        self.create_student_list()
        self.create_buttons()
        
        # 初始加载学生列表
        self.refresh_student_list()
    
    def create_database(self):
        """创建数据库和表"""
        conn = sqlite3.connect('students.db')
        c = conn.cursor()
        c.execute('''CREATE TABLE IF NOT EXISTS students
                     (id INTEGER PRIMARY KEY AUTOINCREMENT,
                      name TEXT NOT NULL,
                      student_id TEXT UNIQUE NOT NULL,
                      gender TEXT,
                      age INTEGER,
                      major TEXT,
                      email TEXT)''')
        conn.commit()
        conn.close()
    
    def create_menu(self):
        """创建菜单栏"""
        menubar = Menu(self.root)
        
        # 文件菜单
        file_menu = Menu(menubar, tearoff=0)
        file_menu.add_command(label="退出", command=self.root.quit)
        menubar.add_cascade(label="文件", menu=file_menu)
        
        # 操作菜单
        action_menu = Menu(menubar, tearoff=0)
        action_menu.add_command(label="添加学生", command=self.show_add_dialog)
        action_menu.add_command(label="刷新列表", command=self.refresh_student_list)
        menubar.add_cascade(label="操作", menu=action_menu)
        
        # 帮助菜单
        help_menu = Menu(menubar, tearoff=0)
        help_menu.add_command(label="关于", command=self.show_about)
        menubar.add_cascade(label="帮助", menu=help_menu)
        
        self.root.config(menu=menubar)
    
    def create_search_frame(self):
        """创建搜索框区域"""
        search_frame = ttk.LabelFrame(self.root, text="搜索学生", padding=(10, 5))
        search_frame.pack(fill=tk.X, padx=10, pady=5)
        
        self.search_var = tk.StringVar()
        search_entry = ttk.Entry(search_frame, textvariable=self.search_var, width=40)
        search_entry.pack(side=tk.LEFT, padx=5)
        
        search_button = ttk.Button(search_frame, text="搜索", command=self.search_students)
        search_button.pack(side=tk.LEFT, padx=5)
        
        clear_button = ttk.Button(search_frame, text="清除", command=self.clear_search)
        clear_button.pack(side=tk.LEFT, padx=5)
        
        # 绑定回车键搜索
        search_entry.bind('<Return>', lambda event: self.search_students())
    
    def create_student_list(self):
        """创建学生列表显示区域"""
        list_frame = ttk.Frame(self.root)
        list_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=5)
        
        # 创建Treeview
        self.tree = ttk.Treeview(list_frame, columns=('name', 'student_id', 'gender', 'age', 'major', 'email'), show='headings')
        
        # 设置列
        self.tree.heading('name', text='姓名')
        self.tree.heading('student_id', text='学号')
        self.tree.heading('gender', text='性别')
        self.tree.heading('age', text='年龄')
        self.tree.heading('major', text='专业')
        self.tree.heading('email', text='邮箱')
        
        # 设置列宽
        self.tree.column('name', width=120)
        self.tree.column('student_id', width=100)
        self.tree.column('gender', width=60)
        self.tree.column('age', width=60)
        self.tree.column('major', width=150)
        self.tree.column('email', width=200)
        
        # 添加滚动条
        scrollbar = ttk.Scrollbar(list_frame, orient=tk.VERTICAL, command=self.tree.yview)
        self.tree.configure(yscroll=scrollbar.set)
        
        # 布局
        self.tree.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        
        # 绑定双击事件编辑学生
        self.tree.bind('<Double-1>', self.edit_selected_student)
    
    def create_buttons(self):
        """创建功能按钮区域"""
        button_frame = ttk.Frame(self.root)
        button_frame.pack(fill=tk.X, padx=10, pady=5)
        
        add_button = ttk.Button(button_frame, text="添加学生", command=self.show_add_dialog)
        add_button.pack(side=tk.LEFT, padx=5)
        
        edit_button = ttk.Button(button_frame, text="编辑学生", command=self.edit_selected_student)
        edit_button.pack(side=tk.LEFT, padx=5)
        
        delete_button = ttk.Button(button_frame, text="删除学生", command=self.delete_selected_student)
        delete_button.pack(side=tk.LEFT, padx=5)
        
        refresh_button = ttk.Button(button_frame, text="刷新列表", command=self.refresh_student_list)
        refresh_button.pack(side=tk.LEFT, padx=5)
        
        exit_button = ttk.Button(button_frame, text="退出", command=self.root.quit)
        exit_button.pack(side=tk.RIGHT, padx=5)
    
    def refresh_student_list(self):
        """刷新学生列表"""
        # 清空当前显示
        for item in self.tree.get_children():
            self.tree.delete(item)
        
        # 从数据库获取学生数据
        conn = sqlite3.connect('students.db')
        c = conn.cursor()
        c.execute("SELECT name, student_id, gender, age, major, email FROM students")
        students = c.fetchall()
        conn.close()
        
        # 插入到Treeview
        for student in students:
            self.tree.insert("", "end", values=student)
    
    def search_students(self):
        """搜索学生"""
        search_term = self.search_var.get().strip()
        conn = sqlite3.connect('students.db')
        c = conn.cursor()
        
        if search_term:
            # 模糊查询
            c.execute("""SELECT name, student_id, gender, age, major, email 
                         FROM students 
                         WHERE name LIKE ? OR student_id LIKE ? OR major LIKE ?""",
                     (f"%{search_term}%", f"%{search_term}%", f"%{search_term}%"))
        else:
            # 显示所有学生
            c.execute("SELECT name, student_id, gender, age, major, email FROM students")
        
        students = c.fetchall()
        conn.close()
        
        # 清空当前显示
        for item in self.tree.get_children():
            self.tree.delete(item)
        
        # 插入查询结果
        for student in students:
            self.tree.insert("", "end", values=student)
    
    def clear_search(self):
        """清除搜索"""
        self.search_var.set("")
        self.refresh_student_list()
    
    def show_add_dialog(self):
        """显示添加学生对话框"""
        self.add_window = tk.Toplevel(self.root)
        self.add_window.title("添加学生")
        self.add_window.grab_set()  # 模态对话框
        
        # 表单框架
        form_frame = ttk.Frame(self.add_window, padding=(10, 5))
        form_frame.pack(fill=tk.BOTH, expand=True)
        
        # 姓名
        ttk.Label(form_frame, text="姓名*:").grid(row=0, column=0, sticky=tk.E, pady=5)
        self.name_var = tk.StringVar()
        ttk.Entry(form_frame, textvariable=self.name_var, width=30).grid(row=0, column=1, pady=5, sticky=tk.W)
        
        # 学号
        ttk.Label(form_frame, text="学号*:").grid(row=1, column=0, sticky=tk.E, pady=5)
        self.id_var = tk.StringVar()
        ttk.Entry(form_frame, textvariable=self.id_var, width=30).grid(row=1, column=1, pady=5, sticky=tk.W)
        
        # 性别
        ttk.Label(form_frame, text="性别*:").grid(row=2, column=0, sticky=tk.E, pady=5)
        self.gender_var = tk.StringVar(value="男")
        ttk.Radiobutton(form_frame, text="男", variable=self.gender_var, value="男").grid(row=2, column=1, sticky=tk.W)
        ttk.Radiobutton(form_frame, text="女", variable=self.gender_var, value="女").grid(row=2, column=1, sticky=tk.W, padx=60)
        
        # 年龄
        ttk.Label(form_frame, text="年龄:").grid(row=3, column=0, sticky=tk.E, pady=5)
        self.age_var = tk.StringVar()
        ttk.Entry(form_frame, textvariable=self.age_var, width=30).grid(row=3, column=1, pady=5, sticky=tk.W)
        
        # 专业
        ttk.Label(form_frame, text="专业:").grid(row=4, column=0, sticky=tk.E, pady=5)
        self.major_var = tk.StringVar()
        ttk.Entry(form_frame, textvariable=self.major_var, width=30).grid(row=4, column=1, pady=5, sticky=tk.W)
        
        # 邮箱
        ttk.Label(form_frame, text="邮箱:").grid(row=5, column=0, sticky=tk.E, pady=5)
        self.email_var = tk.StringVar()
        ttk.Entry(form_frame, textvariable=self.email_var, width=30).grid(row=5, column=1, pady=5, sticky=tk.W)
        
        # 按钮框架
        button_frame = ttk.Frame(self.add_window, padding=(10, 5))
        button_frame.pack(fill=tk.X)
        
        ttk.Button(button_frame, text="提交", command=self.add_student).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="取消", command=self.add_window.destroy).pack(side=tk.LEFT, padx=5)
    
    def add_student(self):
        """添加学生到数据库"""
        # 获取输入数据
        name = self.name_var.get().strip()
        student_id = self.id_var.get().strip()
        gender = self.gender_var.get()
        age = self.age_var.get().strip()
        major = self.major_var.get().strip()
        email = self.email_var.get().strip()
        
        # 验证输入
        if not name or not student_id:
            messagebox.showerror("错误", "姓名和学号是必填项!")
            return
        
        # 插入数据库
        try:
            conn = sqlite3.connect('students.db')
            c = conn.cursor()
            c.execute("INSERT INTO students (name, student_id, gender, age, major, email) VALUES (?, ?, ?, ?, ?, ?)",
                     (name, student_id, gender, age or None, major, email))
            conn.commit()
            messagebox.showinfo("成功", "学生信息添加成功!")
            self.add_window.destroy()
            self.refresh_student_list()
        except sqlite3.IntegrityError:
            messagebox.showerror("错误", "学号已存在!")
        except Exception as e:
            messagebox.showerror("错误", f"添加失败: {str(e)}")
        finally:
            conn.close()
    
    def edit_selected_student(self, event=None):
        """编辑选中的学生"""
        selected_item = self.tree.selection()
        if not selected_item:
            messagebox.showwarning("警告", "请先选择要编辑的学生!")
            return
        
        # 获取当前选中学生的信息
        student_data = self.tree.item(selected_item)['values']
        
        # 创建编辑窗口
        self.edit_window = tk.Toplevel(self.root)
        self.edit_window.title("编辑学生信息")
        self.edit_window.grab_set()  # 模态对话框
        
        # 表单框架
        form_frame = ttk.Frame(self.edit_window, padding=(10, 5))
        form_frame.pack(fill=tk.BOTH, expand=True)
        
        # 姓名
        ttk.Label(form_frame, text="姓名*:").grid(row=0, column=0, sticky=tk.E, pady=5)
        self.edit_name_var = tk.StringVar(value=student_data[0])
        ttk.Entry(form_frame, textvariable=self.edit_name_var, width=30).grid(row=0, column=1, pady=5, sticky=tk.W)
        
        # 学号
        ttk.Label(form_frame, text="学号*:").grid(row=1, column=0, sticky=tk.E, pady=5)
        self.edit_id_var = tk.StringVar(value=student_data[1])
        ttk.Entry(form_frame, textvariable=self.edit_id_var, width=30, state='readonly').grid(row=1, column=1, pady=5, sticky=tk.W)
        
        # 性别
        ttk.Label(form_frame, text="性别*:").grid(row=2, column=0, sticky=tk.E, pady=5)
        self.edit_gender_var = tk.StringVar(value=student_data[2])
        ttk.Radiobutton(form_frame, text="男", variable=self.edit_gender_var, value="男").grid(row=2, column=1, sticky=tk.W)
        ttk.Radiobutton(form_frame, text="女", variable=self.edit_gender_var, value="女").grid(row=2, column=1, sticky=tk.W, padx=60)
        
        # 年龄
        ttk.Label(form_frame, text="年龄:").grid(row=3, column=0, sticky=tk.E, pady=5)
        self.edit_age_var = tk.StringVar(value=student_data[3] if student_data[3] else "")
        ttk.Entry(form_frame, textvariable=self.edit_age_var, width=30).grid(row=3, column=1, pady=5, sticky=tk.W)
        
        # 专业
        ttk.Label(form_frame, text="专业:").grid(row=4, column=0, sticky=tk.E, pady=5)
        self.edit_major_var = tk.StringVar(value=student_data[4] if student_data[4] else "")
        ttk.Entry(form_frame, textvariable=self.edit_major_var, width=30).grid(row=4, column=1, pady=5, sticky=tk.W)
        
        # 邮箱
        ttk.Label(form_frame, text="邮箱:").grid(row=5, column=0, sticky=tk.E, pady=5)
        self.edit_email_var = tk.StringVar(value=student_data[5] if student_data[5] else "")
        ttk.Entry(form_frame, textvariable=self.edit_email_var, width=30).grid(row=5, column=1, pady=5, sticky=tk.W)
        
        # 按钮框架
        button_frame = ttk.Frame(self.edit_window, padding=(10, 5))
        button_frame.pack(fill=tk.X)
        
        ttk.Button(button_frame, text="更新", command=lambda: self.update_student(student_data[1])).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="取消", command=self.edit_window.destroy).pack(side=tk.LEFT, padx=5)
    
    def update_student(self, original_id):
        """更新学生信息"""
        # 获取修改后的数据
        name = self.edit_name_var.get().strip()
        student_id = self.edit_id_var.get().strip()
        gender = self.edit_gender_var.get()
        age = self.edit_age_var.get().strip()
        major = self.edit_major_var.get().strip()
        email = self.edit_email_var.get().strip()
        
        # 验证输入
        if not name or not student_id:
            messagebox.showerror("错误", "姓名和学号是必填项!")
            return
        
        # 更新数据库
        try:
            conn = sqlite3.connect('students.db')
            c = conn.cursor()
            c.execute("""UPDATE students SET 
                        name=?, gender=?, age=?, major=?, email=?
                        WHERE student_id=?""",
                    (name, gender, age or None, major, email, original_id))
            conn.commit()
            messagebox.showinfo("成功", "学生信息更新成功!")
            self.edit_window.destroy()
            self.refresh_student_list()
        except Exception as e:
            messagebox.showerror("错误", f"更新失败: {str(e)}")
        finally:
            conn.close()
    
    def delete_selected_student(self):
        """删除选中的学生"""
        selected_item = self.tree.selection()
        if not selected_item:
            messagebox.showwarning("警告", "请先选择要删除的学生!")
            return
        
        student_id = self.tree.item(selected_item)['values'][1]
        student_name = self.tree.item(selected_item)['values'][0]
        
        if messagebox.askyesno("确认", f"确定要删除学生 {student_name} (学号: {student_id}) 吗?"):
            try:
                conn = sqlite3.connect('students.db')
                c = conn.cursor()
                c.execute("DELETE FROM students WHERE student_id=?", (student_id,))
                conn.commit()
                messagebox.showinfo("成功", "学生信息已删除!")
                self.refresh_student_list()
            except Exception as e:
                messagebox.showerror("错误", f"删除失败: {str(e)}")
            finally:
                conn.close()
    
    def show_about(self):
        """显示关于信息"""
        messagebox.showinfo("关于", "学生信息管理系统\n版本 1.0\n\n使用Python和Tkinter开发")
    if __name__ == "__main__":
         root = tk.Tk()
        app = StudentManagementSystem(root)
        root.mainloop()
