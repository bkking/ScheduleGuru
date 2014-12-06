ScheduleGuru
============

from flask import render_template, request
from app import app
from schedule_api import get_terms, get_schools, get_subjects_by_school, get_classes_by_search, get_class_by_number

@app.route('/')
def index():
    options = {}
    try:
        options['title'] = "University of Michigan Schedule Guru"
    except:
        options['api_error'] = True

    return render_template('index.html', **options)

@app.route('/course-guide/')
def courseguide():
    options = {}
    try:
        options['title'] = "Course Guide - University of Michigan Schedule Guru"
        options['terms'] = get_terms()
    except:
        options['api_error'] = True

    return render_template('course-guide.html', **options)
	
@app.route('/search/')
def search():
    options = {}
    try:
        options['title'] = "Search Results - University of Michigan Schedule Guru"
        searchoption = request.args.get('options', '')
        termcode = request.args.get('term', '')
        keyword = request.args.get('keyword','')
        schoolcode = request.args.get('schoolcode', None)
        page = request.args.get('page', 1)

        options['term'] = termcode
        options['searchoption'] = searchoption.lower()
        options['keyword'] = keyword
        options['schoolcode'] = schoolcode
        options['page'] = page

        if searchoption.lower() == "school":
            if schoolcode is None:
                schools = get_schools(termcode)
                tempschools = []
                for school in schools:
                    if school['SchoolDescr'].find(keyword) > -1:
                        tempschools.append(school)
                options['schools'] = tempschools
            else:
                options['subjects'] = get_subjects_by_school(termcode, schoolcode)
        elif searchoption.lower() == "subject":
            pageddata = get_classes_by_search(termcode, keyword, page)
            if not pageddata is None:
                options['classes'] = pageddata['classes']
                options['start'] = pageddata['start']
                options['end'] = pageddata['end']
                options['count'] = pageddata['count']
                options['totalpages'] = pageddata['totalpages']
                if pageddata['currentpage'] > 1:
                    options['prevpage'] = pageddata['currentpage'] - 1
                if pageddata['currentpage'] < pageddata['totalpages']:
                    options['nextpage'] = pageddata['currentpage'] + 1
                

        
    except Exception as error:
        options['api_error'] = True
        options['api_error_message'] = error.message

    return render_template('search.html', **options)

@app.route('/class-detail/')
def classdetail():
    options = {}
    try:
        options['title'] = "Class Detail - University of Michigan Schedule Guru"
        termcode = request.args.get('term', '')
        classnumber = request.args.get('classnumber', None)

        options['term'] = termcode
        options['classnumber'] = classnumber
        options['classdetail'] = get_class_by_number(termcode, classnumber)
            
    except:
        options['api_error'] = True

    return render_template('class-detail.html', **options)
	
	
@app.errorhandler(404)
def page_not_found(error):
    options = {}
    options['title'] = "404 Page Not Found - University of Michigan Schedule Guru"
    return render_template('page_not_found.html', **options), 404

