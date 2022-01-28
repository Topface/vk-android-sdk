Инструкция
Для адаптации sdk под проект TwoSteps

1 - удалить неиспользуемые подпроекты

2 - добавить типы сборки, такие же как в Twosteps
buildTypes {
    debug {
        testCoverageEnabled = true
    }
    release {
        minifyEnabled false
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
    qa {
        minifyEnabled false
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
    staging {
        matchingFallbacks = ["release", "debug", "qa"]
    }
}

3 - добавить возможность отправлять id при логине
package com.vk.api.sdk.auth.VKAuthManager
fun login(activity: Activity, appId: Int, scopes: Collection<VKScope>) {
    val params = VKAuthParams(appId, scope = prepareScopes(scopes))
    if (VKUtils.isIntentAvailable(activity, VK_APP_AUTH_ACTION, null, VK_APP_PACKAGE_ID)) {
        startAuthActivity(activity, params)
    } else {
        // start dialog with webview
        startInternalAuthActivity(activity, params)
    }
}

package com.vk.api.sdk.VK
fun login(activity: Activity, appId: Int = getAppId(activity), scopes: Collection<VKScope> = emptySet()) {
    authManager.login(activity, appId, scopes)
}

4 - null Safety
com.vk.api.sdk.ui.VKCaptchaActivity
private fun loadImage() {
val url = intent.getStringExtra(KEY_URL)
        VKScheduler.networkExecutor.submit {
            url?.let {
                val data = VKLoader.load(url)
                data?.let {
                    displayImage(BitmapFactory.decodeByteArray(data, 0, data.size))
                }
            }
        }
    }

com.vk.api.sdk.ui.VKWebViewAuthActivity
private fun loadUrl() {
    try {
        val urlToLoad = if (needValidationResult()) intent.getStringExtra(VK_EXTRA_VALIDATION_URL)
    else {
        val uri = Uri.parse("https://oauth.vk.com/authorize").buildUpon()
        val params = getUrlParams()
        for ((key, value) in params) {
            uri.appendQueryParameter(key, value)
        }
    uri.build().toString()
    } 
        urlToLoad?.let { webView.loadUrl(urlToLoad) }
    } catch (e: Exception) {
                setResult(RESULT_CANCELED)
                finish()
            }
    }

private fun handleUrl(url: String?): Boolean {
if (url == null) return false
            if (needValidationResult()) {
                val uri = Uri.parse(url.replace("#", "?"))
                if (uri.getQueryParameter("success") != null) {
                    handleSuccess(uri)
                } else if (uri.getQueryParameter("cancel") != null) {
                    notifyLockerAndFinish()
                }
                return false
            } else {
                if (redirectUrl == null || !url.startsWith(redirectUrl as String)) {
                    return false
                }
                val intent = Intent(VK_RESULT_INTENT_NAME)
                val extraData = url.substring(url.indexOf("#") + 1)
                intent.putExtra(VKAuthManager.VK_EXTRA_TOKEN_DATA, extraData)
                val resultParams = VKUtils.explodeQueryString(extraData)
                val resultCode = if (resultParams != null
                    && (resultParams.containsKey("error")
                        || resultParams.containsKey("cancel"))) RESULT_CANCELED else RESULT_OK
                setResult(resultCode, intent)
                notifyLockerAndFinish()
            }
            return true
        }

5 - обновить пути зависимых проектов с учётом TwoSteps
implementation project(":lib:vk-android-sdk:id")