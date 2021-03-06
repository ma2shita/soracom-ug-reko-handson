<template>
  <v-row align="start" justify="center">
    <v-col cols="12" sm="8" md="6">
      <v-alert type="error" v-if="errorMessage">
        {{errorMessage}}
      </v-alert>
      <v-alert type="success" v-if="successMessage">
        {{successMessage}}
      </v-alert>
      <v-card class="elevation-12">
        <v-toolbar color="indigo lighten-3" dark flat>
          <v-toolbar-title>顔画像認識</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-dialog v-model="settingDialogRecognition" persistent max-width="600px">
            <template v-slot:activator="{ on }">
              <v-btn icon dark v-on="on">
                <v-icon>mdi-account-question</v-icon>
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">顔画像認識設定</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-textarea v-model="updateApiEndpoint" label="APIエンドポイント" auto-grow rows="1" row-height="15" clearable></v-textarea>
                  <v-textarea v-model="updateApiKey" label="APIキー" auto-grow rows="1" row-height="15" clearable></v-textarea>
                  <v-select
                    v-model="threshold"
                    :items="[10,20,30,40,50,60,70,80,90,100]"
                    label="しきい値"
                    clearable>
                  </v-select>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-row>
                  <v-col>
                    <v-btn color="warning" @click="settingCancel()" block large><v-icon left>mdi-close-box</v-icon>キャンセル</v-btn>
                  </v-col>
                  <v-col>
                    <v-btn color="success" @click="settingSave()" block large><v-icon left>mdi-content-save</v-icon>保存</v-btn>
                  </v-col>
                </v-row>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
        <v-card-text>
          <v-form>
            <v-file-input show-size label="画像ファイルを選択" accept="image/*" @change="onFileChange"></v-file-input>
            <div class="preview-item" style="text-align: center;">
              <img
                v-show="uploadedImage"
                class="preview-item-file"
                :src="uploadedImage"
                alt=""
                width="200px"
              />
            </div>
            <div v-show="noTarget"><v-text-field label="認識結果" v-model='noTarget' outlined editable></v-text-field></div>
            <div v-show="faceMatch">
              <v-simple-table>
                <template v-slot:default>
                  <thead>
                    <tr>
                      <th>マッチした人物</th>
                      <th>信頼度</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr>
                      <td>{{faceMatch}}</td>
                      <td>{{faceMatchConf}}</td>
                    </tr>
                  </tbody>
                </template>
              </v-simple-table>
            </div>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
            <v-btn color="warning" class="ma-2 white--text" x-large @click="execRekognition" v-if="uploadedImage">
              <v-icon dark>mdi-cloud-upload</v-icon>
              &nbsp;顔画像認識実行
            </v-btn>
          <v-spacer />
        </v-card-actions>
      </v-card>
      <v-overlay :value="overlay">
        <v-progress-circular indeterminate size="64"></v-progress-circular>
      </v-overlay>
    </v-col>
  </v-row>
</template>

<script>
  import axios from 'axios';

  export default {
    name: 'SearchFaces',

    data: () => ({
      uploadedImage: '',
      noTarget: false,
      overlay: false,
      successMessage: '',
      errorMessage: '',
      settingDialogRecognition: false,
      faceMatch: false,
      faceMatchConf: null,
      updateApiEndpoint: '',
      updateApiKey: ''
    }),
    created() {
      // Configミックスインの情報を設定
      this.apiEndpoint = this.config.searchConfig.apiEndpoint;
      this.apiKey = this.config.searchConfig.apiKey;
      this.threshold = this.config.searchConfig.threshold;
      // SORACOMメタデータサービスにアクセスしAPI接続情報を取得
      this.getSoracomMetadata();
    },
    methods: {
      /**
       * 画像変更時処理
       */
      onFileChange(e) {
        this.noTarget = false;
        if (e != null) {
          this.createImage(e);
          this.uploadedImage = false;
          this.faceMatch = false;
        } else {
          this.uploadedImage = false;
          this.faceMatch = false;
        }
        this.errorMessage = '';
      },
      /**
       * 画像取得処理
       */
      createImage(file) {
        const reader = new FileReader();
        reader.onload = e => {
          this.uploadedImage = e.target.result;
        };
        reader.readAsDataURL(file);
      },
      /**
       * 顔認識実行処理
       */
      async execRekognition() {
        // 設定情報入力チェック
        this.errorMessage = ''
        if (!this.apiEndpoint) {
          this.errorMessage = "設定画面でAPIエンドポイントを入力してください";
        } else if (!this.apiKey) {
          this.errorMessage = "設定画面でAPIキーを入力してください";
        }else if (!this.threshold) {
          this.errorMessage = "設定画面でしきい値を入力してください";
        }
        // エラー時は処理終了
        if (this.errorMessage) {
          return
        } else {
          this.overlay = true;
        }
        // 実行時パラメータ構築
        const querydata = {
          'image_base64str': this.uploadedImage,
          'threshold': this.threshold,
        };
        const config = {headers: {
          'Content-Type': 'application/json',
          'x-api-key': this.apiKey,
        }};
        // API呼び出し
        axios
          .post(this.apiEndpoint, JSON.stringify(querydata), config)
          .then(response => {
              const faceMatches = response.data.payloads.FaceMatches;
              if (faceMatches.length == 0) {
                this.noTarget = '人物を特定できませんでした'
              } else {
                this.createAuthenticationData(faceMatches[0]);
              }
              this.overlay = false;
          })
          .catch(error => {
            this.errorMessage = error;
            this.overlay = false;
          });
      },
      /**
       * SORACOMメタデータ(userdata)取得
       */
      getSoracomMetadata() {
        // API接続情報の初期化
        this.apiEndpoint = '';
        this.apiKey = '';
        // HTTP非同期通信
        axios
          .get("http://metadata.soracom.io/v1/userdata", { timeout : 1000 })
          .then(response => {
            if (response.status == 200) {
              // SIMグループから取得したuserdataを内部変数に保存
              this.apiEndpoint = response.data.apiEndpoint;
              this.apiKey = response.data.apiKey;
              this.updateApiEndpoint = this.apiEndpoint;
              this.updateApiKey = this.apiKey;
            } else {
              console.log('レスポンスステータス：' + response.status);
              this.errorMessage = 'メタデータの取得に失敗しました。API接続情報を手動で入力してください。';
            }
          })
          .catch(error => {
            // メタデータ取得時にエラーが発生した場合はエンドポイント・ApiKeyクリア
            this.errorMessage = 'メタデータの取得に失敗しました。API接続情報を手動で入力してください：' + error;
          });
      },
      /**
       * 顔認識結果編集処理
       */
      createAuthenticationData(faceMatch) {
        this.faceMatch = faceMatch.Face.ExternalImageId;
        this.faceMatchConf = Math.round(faceMatch.Similarity * 100) / 100 + '%';
      },
      /**
       * 設定変更保存
       */
      settingSave() {
        this.apiEndpoint = this.updateApiEndpoint;
        this.apiKey = this.updateApiKey;
        this.settingDialogRecognition = false;
      },
      /**
       * 設定変更キャンセル
       */
      settingCancel() {
        this.updateApiEndpoint = this.apiEndpoint;
        this.updateApiKey = this.apiKey;
        this.settingDialogRecognition = false;
      }
    }
  };
</script>>