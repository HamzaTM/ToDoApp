# # ToDoApp
# Summary 
- [Overview](#Overview)
- [Defining a Task](#Defining-a-Tasks)
- [Add task function](#Add-task-function)
- [How to hide and show the finished and the pending tasks](#)
- [Save tasks](#Save-tasks)
- [Open tasks](#Open-tasks)
- [Change the content of an item](#Change-the-content-of-an-item)
- [MVC model](#MVC-model)
- [THE END](#THE-END)

## Overview
- In this homework 4 we are trying to create a task manager app to help us to manage your daily tasks easily.

![1](https://user-images.githubusercontent.com/80457241/150691104-0189812f-c34d-410b-b57e-6a6cfb8abeb9.PNG)


## Defining a Task
Now we are going to show how to define a task

A Task is defined by the following attributes:
- A description: stating the text and goal for the task like (Buying the milk).
- A finished Boolean indicating if the task is Finished or due.
- A Tag category to show the class of the task which is reduced to the following values:
  - Work
  - Life
  - Other
- Finally, a task should have a DueDate which stores the Date planned for the date.

The result should look like this:
![2](https://user-images.githubusercontent.com/80457241/150691105-6acb7c37-1498-435e-b7f4-bee23b2301d4.PNG)

Now , we are adding setters :
- date setter 
```cpp
void addTaskDialog::setdate(int y,int m, int d){
    QDate da;

    da.setDate(y,m,d);
    ui->dateEdit->setDate(da);
}
```
- description setter
```cpp
void addTaskDialog::setdesc(QString a){
    ui->lineEdit->setText(a);
}
```
- tag setter
```cpp
void addTaskDialog::settag(QString a){
    ui->comboBox->setCurrentText(a);
}
```
- checkbox setter
```cpp
void addTaskDialog::setf(bool a){
    ui->checkBox->setChecked(a);
}
```
- gettext function
```cpp
QString addTaskDialog::getText(){

    QString a= ui->lineEdit->text() + " Due : " + ui->dateEdit->text() + "  Tag : " + ui->comboBox->currentText() +"";
    return a;
}
``````
Here we are defining a minimum date  :
```cpp
    QDate date =QDate::currentDate();
    ui->dateEdit->setMinimumDate(date);
    ui->dateEdit->setDate(date);
```
## Add task function
We are adding a slot to our header::
```cpp
private slots :
  void on_action_Add_triggered();
```
```cpp
     //first we call the task dialog
     addTaskDialog dialog;
     auto reply= dialog.exec();
     if(reply == addTaskDialog::Accepted)
     {
         //we save the task on a text string using the gettext function
         QString text= dialog.getText();
         // now based on the date and the checkbox status we will decide on wich listwidget we are going to add our task as an item
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             ui->listWidget->addItem(text);
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             ui->lw2->addItem(text);
         }else if(dialog.isChecked()){
             ui->lw3->addItem(text);
         }
     }
```
## How to hide and show the finished and the pending tasks 
The user can either hide or show the pending and finished tasks.
We are adding the following slots to our header:
```cpp
private slots:
    void on_action_View_pending_tasks_toggled(bool arg1);
    void on_action_View_finished_tasks_toggled(bool arg1);
```
- The execution of on_action_View_pending_tasks_toggled slot
```cpp
void ToDoApp::on_action_View_pending_tasks_toggled(bool arg1)
{
    if(arg1){
        ui->lw2->setVisible(true);
    }else{
        ui->lw2->setVisible(false);
    }
}
```
- the execution of on_action_View_finished_tasks_toggled slot
```cpp
void ToDoApp::on_action_View_finished_tasks_toggled(bool arg1)
{
    if(arg1){
        ui->lw3->setVisible(true);
    }else{
        ui->lw3->setVisible(false);
    }
}
```
This two lines will our widgets invisible by default we have to add them to our  constructor 
```cpp
   ui->lw2->setVisible(false);
   ui->lw3->setVisible(false);
```
## Save tasks
We will create a save function that help us to save our tasks after closing our application :

```cpp
protected:
    void closeEvent(QCloseEvent* e) override;
```
The tasks will be saved on a .txt file :
```cpp
void ToDoApp::closeEvent(QCloseEvent* e){
    
    QFile file("C:/Users/hamza/C++/done.txt");
    if(file.open(QIODevice::ReadWrite | QIODevice::Text)){
        QTextStream out(&file);
        
        for(int i=0;i<ui->listWidget->count();i++)
        {
            out << "1"<< ui->listWidget->item(i)->text() << Qt::endl;
        }
        for(int i=0;i<ui->lw2->count();i++)
        {
            out << "2"<< ui->lw2->item(i)->text() << Qt::endl;
        }
        for(int i=0;i<ui->lw3->count();i++)
        {
            out << "3"<< ui->lw3->item(i)->text() << Qt::endl;
        }
        file.close();
    }
}
```
## Open tasks
 This code make our  tasks entered to our application  remains there to use it when ever we want  
```cpp
      QFile file("C:/Users/hamza/C++/done.txt");

      if (!file.open(QIODevice::ReadOnly | QIODevice::Text))
            return;
      while (!file.atEnd()) {
          QString line = file.readLine();
          if(line.at(0)=="1"){
                ui->listWidget->addItem(line.mid(1,line.size()));
          }else if(line.at(0)=="2"){
              ui->lw2->addItem(line.mid(1,line.size()));
          }else if(line.at(0)=="3"){
              ui->lw3   ->addItem(line.mid(1,line.size()));
          }
      }
```
## How to change the content of an task
We need to link the list widget to a function that is automatically executed if a task is double clicked :
```cpp
connect(ui->listWidget, SIGNAL(itemDoubleClicked(QListWidgetItem*)), this,SLOT(v1()));
connect(ui->lw2, SIGNAL(itemDoubleClicked(QListWidgetItem*)), this,SLOT(v2()));
connect(ui->lw3, SIGNAL(itemDoubleClicked(QListWidgetItem*)), this,SLOT(v3()));
```
We have to add these three slots to the header :
```cpp
private slots:    
    void v1();
    void v2();
    void v3();
```
The execution of the slots v1():
```cpp
void ToDoApp::v1(){
    addTaskDialog dialog;
    int i=1;
    
    QString task;
    // declare the new task
    QListWidgetItem *a=ui->listWidget->currentItem();
    // now we split the task 
    QStringList list = a->text().split(QRegularExpression("\\W+"), Qt::SkipEmptyParts);
    task+=list[0];
    
    while(list[i]!="Due"){
         task+=" "+list[i];
         i++;
    }
    // set the date
   dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    // set the description
    dialog.setdesc(task);
    //set the tag
    dialog.settag(list[i+5]);
    auto reply= dialog.exec();
    if(reply == addTaskDialog::Accepted)
     {
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             ui->listWidget->addItem(text);
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             ui->lw2->addItem(text);
         }else if(dialog.isChecked()){
             ui->lw3->addItem(text);
         }
         // delete the previous item so we won't have a duplicated one 
         delete a ;
     }
}
```
Same for the second slot :
```cpp
void ToDoApp::v2(){
    addTaskDialog dialog;
    int i=1;
    
    QString task;
    // declare the new task
    QListWidgetItem *a=ui->listWidget->currentItem();
    // now we split the task 
    QStringList list = a->text().split(QRegularExpression("\\W+"), Qt::SkipEmptyParts);
    task+=list[0];
    
    while(list[i]!="Due"){
         task+=" "+list[i];
         i++;
    }
    
    // set the date
    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    // set the description
    dialog.setdesc(task);
    //set the check box for finished tasks to true
    dialog.setf(true);
    //set the tag
    dialog.settag(list[i+5]);
    auto reply= dialog.exec();
    if(reply == addTaskDialog::Accepted)
     {
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             ui->listWidget->addItem(text);
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             ui->lw2->addItem(text);
         }else if(dialog.isChecked()){
             ui->lw3->addItem(text);
         }
         // delete the previous item so we won't have a duplicated one 
         delete a ;
     }
}
```
The results :
- step 1: create a task
![1](https://user-images.githubusercontent.com/80457241/150692828-8dcc7ff1-9b08-4ae6-b443-2f01bc11a3a8.PNG)

- step 2: double click on the task

![3](https://user-images.githubusercontent.com/80457241/150693085-784ddbbd-2922-418c-83d5-7be6908db0c2.PNG)



 - step 3: change the attributes
![4](https://user-images.githubusercontent.com/80457241/150693182-1725f878-6e42-4028-8be3-a7e75394a251.PNG)



- step 4: save the changes
![6](https://user-images.githubusercontent.com/80457241/150693307-6db4fa02-4109-4ea9-8349-7c4c8cd81ccc.png)



## MVC model

With Models we are going to make another ToDo App

- With the Qt Designer and using QListView we are going to create a form :
![5](https://user-images.githubusercontent.com/80457241/150693372-09ed23f0-fdaa-40bf-977e-2c509b8a4f10.PNG)


- QStringModel  help us to create models : 
  - We put this code to our header
 ```cpp
    QStringListModel *model1;
    QStringListModel *model2;
    QStringListModel *model3;
    
    QStringList Todaytasks;
    QStringList Finishedtasks;
    QStringList Pendingtasks;
 ```
 ```cpp
    model1 = new QStringListModel();
    model2 = new QStringListModel();
    model3 = new QStringListModel();
    ```
  - After that we need to set the model to the view
  ```cpp
     model1->setStringList(Todaytasks);
     ui->lw1->setModel(model1);


     model2->setStringList(Pendingtasks);
     ui->lw2->setModel(model2);


     model3->setStringList(Finishedtasks);
     ui->lw3->setModel(model3);
  ```
 -  Add function
 ```cpp
 void ToDoApp::on_action_Add_triggered()
 {
     addTaskDialog dialog;
     auto reply= dialog.exec();
     if(reply == addTaskDialog::Accepted)
     {
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             Todaytasks.append(text);
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             Pendingtasks.append(text);
         }else if(dialog.isChecked()){
             Finishedtasks.append(text);
         }
     }

     model1->setStringList(Todaytasks);
     ui->lw1->setModel(model1);

     model2->setStringList(Pendingtasks);
     ui->lw2->setModel(model2);


     model3->setStringList(Finishedtasks);
     ui->lw3->setModel(model3);
 }
 ```
 - Tthe execution of the new open and save function
   - Save :
  ```cpp
   void ToDoApp::closeEvent(QCloseEvent* e){
    QFile file("C:/Users/hamza/Desktop/done.txt");
    if(file.open(QIODevice::ReadWrite | QIODevice::Text)){
        QTextStream out(&file);
        for(int i=0;i<Todaytasks.size();i++)
        {
            out << "1"<< Todaytasks.at(i)<< Qt::endl;
        }
        for(int i=0;i<Pendingtasks.size();i++)
        {
            out << "2"<< Pendingtasks.at(i) << Qt::endl;
        }
        for(int i=0;i<Finishedtasks.size();i++)
        {
            out << "3"<< Finishedtasks.at(i) << Qt::endl;
        }
        file.close();
    }
   }
   ```
   - Open :
     - this will be added to the constructor
   
   ```cpp
      QFile file("C:/Users/hamza/Desktop/done.txt");

      if (!file.open(QIODevice::ReadOnly | QIODevice::Text))
            return;

      while (!file.atEnd()) {
          QString line = file.readLine();
          if(line.at(0)=="1"){
                Todaytasks.append(line.mid(1,line.size()));
          }else if(line.at(0)=="2"){
              Pendingtasks.append(line.mid(1,line.size()));
          }else if(line.at(0)=="3"){
              Finishedtasks.append(line.mid(1,line.size()));
          }
      }
      
      model1->setStringList(Todaytasks);
      ui->lw1->setModel(model1);

      model2->setStringList(Pendingtasks);
      ui->lw2->setModel(model2);

      model3->setStringList(Finishedtasks);
      ui->lw3->setModel(model3);
   ```
 - Change content of a task
   - We connect the lists to the slots that perform these actions
  ```cpp
      connect(ui->lw1, SIGNAL(doubleClicked(QModelIndex)), this,SLOT(v1()));
      connect(ui->lw2, SIGNAL(doubleClicked(QModelIndex)), this,SLOT(v2()));
      connect(ui->lw3, SIGNAL(doubleClicked(QModelIndex)), this,SLOT(v3()));
   ```
   - The execution of v1() :
   ```cpp
   void ToDoApp::v1(){
    addTaskDialog dialog;
    int i=1;
    QString task;
    QModelIndex index=ui->lw1->currentIndex();
    QString a=Todaytasks.at(index.row());
    QStringList list = a.split(QRegularExpression("\\W+"), Qt::SkipEmptyParts);
    task+=list[0];

    while(list[i]!="Due"){
         task+=" "+list[i];
         i++;
    }

    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    dialog.setdesc(task);
    dialog.settag(list[i+5]);
    auto reply= dialog.exec();


     if(reply == addTaskDialog::Accepted)
     {
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             Todaytasks.append(text);
             Todaytasks.removeAt(index.row());

         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             Pendingtasks.append(text);
             Todaytasks.removeAt(index.row());
         }else if(dialog.isChecked()){
             Finishedtasks.append(text);
             Todaytasks.removeAt(index.row());
         }

     }

     model1->setStringList(Todaytasks);
     ui->lw1->setModel(model1);

     model2->setStringList(Pendingtasks);
     ui->lw2->setModel(model2);


     model3->setStringList(Finishedtasks);
     ui->lw3->setModel(model3);

   } 
   ```
   - The execution of v2() :
   ```cpp
   void ToDoApp::v2 (){
    addTaskDialog dialog;
    int i=1;
    QString task;
    QModelIndex index=ui->lw2->currentIndex();
    QString a=Pendingtasks.at(index.row());
    QStringList list = a.split(QRegularExpression("\\W+"), Qt::SkipEmptyParts);
    task+=list[0];
    while(list[i]!="Due"){
         task+=" "+list[i];
         i++;
    }
    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    dialog.setdesc(task);
    dialog.settag(list[i+5]);
    auto reply= dialog.exec();
    if(reply == addTaskDialog::Accepted){
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             Todaytasks.append(text);
             Pendingtasks.removeAt(index.row());
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             Pendingtasks.append(text);
   Pendingtasks.removeAt(index.row());
         }else if(dialog.isChecked()){
             Finishedtasks.append(text);
             Pendingtasks.removeAt(index.row());
         }


    }

    model1->setStringList(Todaytasks);
    ui->lw1->setModel(model1);

    model2->setStringList(Pendingtasks);
    ui->lw2->setModel(model2);


    model3->setStringList(Finishedtasks);
    ui->lw3->setModel(model3);
    }
   ```
   - The execution of v3()
   ```cpp
    void ToDoApp::v3(){
    addTaskDialog dialog;
    int i=1;
    QString task;
    QModelIndex index=ui->lw3->currentIndex();
    QString a=Finishedtasks.at(index.row());
    QStringList list = a.split(QRegularExpression("\\W+"), Qt::SkipEmptyParts);
    task+=list[0];
    while(list[i]!="Due"){
         task+=" "+list[i];
         i++;
    }
    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    dialog.setdesc(task);
    dialog.setf(true);
    dialog.settag(list[i+5]);
    auto reply= dialog.exec();
    if(reply == addTaskDialog::Accepted){
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             Todaytasks.append(text);
              Finishedtasks.removeAt(index.row());
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             Pendingtasks.append(text);
              Finishedtasks.removeAt(index.row());
         }else if(dialog.isChecked()){

         }

    }

    model1->setStringList(Todaytasks);
    ui->lw1->setModel(model1);

    model2->setStringList(Pendingtasks);
    ui->lw2->setModel(model2);

    model3->setStringList(Finishedtasks);
    ui->lw3->setModel(model3);
    }
   ```
# Conclusion
after working on this rapport i learned how to create an app to help you to manage your time easily
