# medicine
Inventory control

from __future__ import print_function
from future.standard_library import install_aliases
install_aliases()

from urllib.parse import urlparse, urlencode
from urllib.request import urlopen, Request
from urllib.error import HTTPError

import json
import os

from flask import Flask
from flask import request
from flask import make_response, jsonify
import gspread
from oauth2client.service_account import ServiceAccountCredentials


# Flask app should start in global layout
app = Flask(__name__)


@app.route('/webhook', methods=['POST'])
def webhook():
    req = request.get_json(silent=True, force=True)
    result = req.get("result")
    parameters = result.get("parameters")
    medicine_name = parameters.get("medicine_name")

    scope = ['https://drive.google.com/open?id=1RseqOYQvR075Phjz3HlKPHnOfEVdo6e4qwOoVV4Es6E']
    
    #ダウンロードしたjsonファイルを同じフォルダに格納して指定する
    credentials = ServiceAccountCredentials.from_json_keyfile_name('xxxxxxxxxx.json', scope)
    gc = gspread.authorize(credentials)
    
    # # 共有設定したスプレッドシートの名前を指定する
    worksheet = gc.open("Google Assistant Commands").get_worksheet(1)

    cell = worksheet.find(medicine_name)

    text = str(cell.value) + str(worksheet.cell(cell.row,cell.col+1).value) + "です"
    r = make_response(jsonify({'speech':text,'displayText':text}))
    r.headers['Content-Type'] = 'application/json'
    return r
    
if __name__ == '__main__':
    port = int(os.getenv('PORT', 5000))

    print("Starting app on port %d" % port)

    app.run(debug=False, port=port, host='0.0.0.0'
