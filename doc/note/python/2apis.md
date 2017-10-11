raw_input('Enter an integer:')    wait for input.
range(1, 5)     generate an array. [1, 2, 3, 4], there is a third parameter as step.
global   declare a variable in a function. the a variable is a global. and it change can display 
            everywhere.
default parameter:   only be the last parameter.
key parameter:   use like this test_function(a=1, b=2). the parameter's value is defined by the key
                          parameter but not the sort.
docStrings: the first line doc String is the function's doc string. can refrence by
                  testFunction.__doc__..        and the in-build function help is to show the doc string
                  in fuction.  like this : help(testFunction)
import : import test. this meaning python will find the test.py in the sys.path. import.. as can shotcut the
             module name
from..import: from sys import path. so we can directly use path to insted sys.path. it must use it as little as
                     possble
__name__: every module must have a name, if it's the single running , it will be __main__.
                otherwise it will be the filename.
dir():  no para display current module's attributes (varible, funciton...). para will be imported module
         del can delete a attribute in a modules.
list: declare like java's array . list = ['','',''].   print list will list the item of list .for in. list.sort()
      list.append() add a item to the list.    del list[0] delete an element.   list[a:b] it's a sublist
tuple: like java's array.  array = ('','','') the length is static.
          with print  : print '%s is %d years old'  % (name, age) %s is a string, %d is a number.   name is a string
          variable. 
map: ab = { '':''}      ab[key]    del ab[key]  
string function: s.startswith('Swa'),    'a' in s,  s.find('war')     s.join(an array)=a string with element in array but
                        split by s
time.strftime('%Y%m%d%H%M%S')  inbuild format current time.     os.system execute commond line.
class:   self ,function must have this parameter , like java's this.
           __init__ like java's constractor.   if it have another parameter except self ,this must use
           classname(paramter) to create an instance.
 file:   f = file('','') first is a file path. second is the type. write or read.
         f.write().
         f.close().
         f.readline().
         if len(f.readline) break
pickling:   can store the py in a file and restore it every time.
try..except: try      except.    
                  custom a exception class MyException(Exception) 
                  raise  throw an exception
==========library
sys module:   sys.argv   arguments list
                     sys.path classpath
                     sys.version
                     sys.stdin                   standard
os module:    os.name              current platform window as NT, linux as posix
                     os.getcwd()            current work menu
                     os.listdir()             specified dir file list
                     os.remove()          delete a file
                     os.system()           execute a shell commond
                     os.linesep             current end char
                     os.path.split(string)   split a file path to a dir and a filename
                     os.path.isfile()  , os.path.isdir,  os.path.exists()
=============
special function:
          __init__(self, ...)   like constructor
          __del__(self) like finallize()
          __str__(self) like toString.
          __lt__()        like equales , it's a series functions use to change + , < ,> operation
          __getitem__(self, key) like hascode()  when use x[key]
          __len__(self)    len()
 
generate list from list:  listtow = [2*i for i in listone if i > 2]
like java ..param: *args. all paramters will be a tuple. ** will be map
exec: execute the python statement which is store as a string.
eval: compute a python expression store as a string.
repr:   same as `` get the print string of object.
 