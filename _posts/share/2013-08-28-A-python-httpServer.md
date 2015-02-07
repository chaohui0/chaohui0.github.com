---
layout: post
title: 一个强大的python http服务器
description: 很好很强大哦
category: share
---

    import glob
    import os.path

    import cherrypy
    from cherrypy.lib.static import serve_file


    class Root:
        def index(self, directory="."):
            html = """<html><body><h2>Here are the files in the selected directory:</h2>
            <a href="index?directory=%s">Up</a><br />
            """ % os.path.dirname(os.path.abspath(directory))

            
            i = 0
            html += '<table border="1" width=1000>'
            html += '<tr>'
            for filename in glob.glob(directory + '/*'):
                html += '<td width="20%">'

                absPath = os.path.abspath(filename)
                if os.path.isdir(absPath):
                    html += '[DIR]<a href="/index?directory=' + absPath + '">' + os.path.basename(filename) + "</a> <br />"
                else:
                    html += '<a href="/download/?filepath=' + absPath + '">' + os.path.basename(filename) + "</a> <br />"
                    if absPath.endswith('jpg') or absPath.endswith('jpeg') or absPath.endswith('png'):
                        html += '<img src="/download/?filepath=' + absPath + '" width=200 height=200/> <br />'

                html += '</td>'
                i += 1
                if i % 5 == 0:
                    html += '</tr><tr>'
            html += '</tr>'
            html += '</table>'

            html += """</body></html>"""
            return html
        index.exposed = True

    class Download:
        def index(self, filepath):
            return serve_file(filepath, "application/x-download", "attachment")
        index.exposed = True

    if __name__ == '__main__':
        root = Root()
        root.download = Download()
        cherrypy.config.update({'server.socket_host':'your ip address', 'server.socket_port':12345})
        cherrypy.quickstart(root)


python的缩进牛逼而又蛋疼！总结完毕
