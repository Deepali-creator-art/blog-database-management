import db_fun
import streamlit as st
from PIL import Image
from db_fun import *
import base64
import pandas as pd
import matplotlib.pyplot as plt
st.set_option('deprecation.showPyplotGlobalUse', False)


#Layout Templates
title_temp="""
<div style="background-color:#00FFFF;padding:10px,margin:10px;">
<h4 style="color:white;text-align:center;">{}</h4>https://meet.google.com/zjk-ophs-chs?pli=1
<h4 style="color:white;text-align:center;">{}</h4>
<h6 style="color:white;text-align:center;">Post Date: {}</h6>
<div style="text-align: center;">
<img src="data:image/jpg;base64,{}" alt="Image" 
style='horizontal-align:center' width="200px" height="200px">
</div>
<br/>
<br/>
<p style='text-align:justify'>{}</p>
</div>
"""
st.subheader("Blog Database management using streamlit")
choice=st.sidebar.selectbox("Select Menu",["Home","Add Post","Search","Manage Blog"])
db_fun.create_table()
if choice=="Home":
    #st.write("View all records")
    result=db_fun.view_all_records()
    #st.write(result)
    for i in result:
        title=i[0]
        author=i[1]
        article=i[2]
        date=i[3]

        # open saved image after uploading
        try:
            b_image = "%s.jpg" %author
            file = open(b_image, "rb")
            contents=file.read()
            b_image = base64.b64encode(contents).decode("utf-8")
            file.close()
        except:
            print("ERROR2: Couldn't open image! Make sure the extension is correct")
        st.markdown(title_temp.format(title,author,date,b_image,article),unsafe_allow_html=True)

elif choice=="Add Post":
    #st.write("Add Post")
    blog_title=st.text_input("Enter the title")
    blog_author=st.text_input("Enter the author name")
    blog_article=st.text_area("Enter the article contents")
    blog_date=st.date_input("Enter published date")
    try:
        # Upload the image
        img_file = st.file_uploader("Upload an image", type=["png", "jpg", "jpeg"])
        image = Image.open(img_file)
        # Save the image to the folder
        image = image.convert('RGB')
        img = '{}.jpg'.format(blog_author)
        image.save(img)
        # Open a file in binary mode
        file = open(img, 'rb').read()
        # We must encode the file to get base64 string
        file = base64.b64encode(file)

    except:
        print("ERROR1: Couldn't open image! Make sure the extension is correct")
    if st.button("Add blog"):
        add_post(blog_title,blog_author,blog_article,blog_date,file)
        st.success(f"Successfully add a blog {blog_title}")
elif choice=="Search":
    st.subheader("Search Articles")
    search_term=st.text_input("Enter the search term")
    choice=st.radio("Search article by fields",('Title','Author'))
    if(choice=='Title'):
        result=db_fun.get_blog_title(search_term)
        #st.write(result)
    if(choice=='Author'):
        result=db_fun.get_blog_author(search_term)
        #st.write(result)
    if st.button("Search"):
        for i in result:
            title=i[0]
            author=i[1]
            article=i[2]
            date=i[3]

            # open saved image after uploading
            try:
                b_image = "%s.jpg" %author
                file = open(b_image, "rb")
                contents=file.read()
                b_image = base64.b64encode(contents).decode("utf-8")
                file.close()
            except:
                print("ERROR2: Couldn't open image! Make sure the extension is correct")
            st.markdown(title_temp.format(title,author,date,b_image,article),unsafe_allow_html=True)

elif choice=="Manage Blog":
    st.subheader("Manage Blog")
    data=view_all_records()

    #st.write(data)
    blog_table=pd.DataFrame(data,columns=['Title','Author','Article','Post Date','Author Image'])

    blog_table['Author Image']=blog_table['Author Image'].apply(lambda x : base64.b64encode(x).decode("utf-8"))
    st.write(blog_table)

    delete_record=st.text_input("Enter author name")
    if st.button('Delete'):
        db_fun.delete_blog(delete_record)
        updated_data = view_all_records()
        blog_table = pd.DataFrame(updated_data, columns=['Title', 'Author', 'Article', 'Post Date', 'Author Image'])
        st.write(blog_table)
    st.subheader("Graphical Visualization")
    title_count=blog_table['Title'].value_counts()
    st.write(title_count)
    title_count.plot(kind='bar')
    st.pyplot()



