package com.munload.vpn

import android.app.Activity
import android.content.Intent
import android.net.VpnService
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.compose.setContent
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.unit.dp

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme {
                Surface(modifier = Modifier.fillMaxSize()) {
                    VpnScreen()
                }
            }
        }
    }
}

@Composable
fun VpnScreen() {
    val context = LocalContext.current

    // Лаунчер для запроса системного разрешения на VPN
    val launcher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == Activity.RESULT_OK) {
            // Пользователь нажал "ОК" в системном окне — запускаем сервис
            val intent = Intent(context, MyVpnService::class.java).apply {
                action = "START"
            }
            context.startService(intent)
        }
    }

    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(text = "Сырое VPN приложение", style = MaterialTheme.typography.headlineSmall)
        
        Spacer(modifier = Modifier.height(32.dp))

        Button(
            onClick = {
                // Проверяем, дано ли разрешение на VPN
                val intent = VpnService.prepare(context)
                if (intent != null) {
                    // Если не дано — показываем системное окно
                    launcher.launch(intent)
                } else {
                    // Если уже дано — запускаем сразу
                    val serviceIntent = Intent(context, MyVpnService::class.java).apply {
                        action = "START"
                    }
                    context.startService(serviceIntent)
                }
            }
        ) {
            Text("Включить VPN")
        }
    }
}
