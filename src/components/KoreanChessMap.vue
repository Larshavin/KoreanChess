<template>
    <div>
        <!-- row : 행 10줄 : 1부터 시작 -->
        <div v-for="i in 10" class="flex not_draggable" @contextmenu.prevent>
            <!-- column : 열 9줄 : 1부터 시작 -->
            <div v-for="j in 9" class="relative p-0 flex justify-content-center align-items-center"
                :style="{ 'height': size + 'px', 'width': size + 'px' }">
                <!-- 실제 기물의 위치 -->
                <div v-if="board[i - 1][j - 1] != 0" class="z-4">
                    <img :src="getPieceImageUrl(board[i - 1][j - 1])" class="z-4 cursor-pointer" :class="{
                        'border-round border-3 border-green-300': isSelected(i - 1, j - 1),
                        'border-round border-3 border-red-300': isLastMovde(i - 1, j - 1),
                        'pointer-events-none': isOnTurn(i - 1, j - 1)
                    }" :style="{ 'height': size * 0.9 + 'px', 'width': size * 0.9 + 'px' }"
                        @click="pathFinding(i - 1, j - 1)" />
                </div>
                <!-- 움직일 수 있는 경로 선택 -->
                <div v-if="isMoveAvailable(i - 1, j - 1)"
                    class="absolute border-circle bg-green-200 border-1 border-green-600 z-5 shadow-2 cursor-pointer"
                    :style="{ 'height': size / 4 + 'px', 'width': size / 4 + 'px' }" @click="move(i - 1, j - 1)">
                </div>
                <!-- 장기판 꾸미기 : 선, 대각선, 엑스표 -->
                <div class="z-0">
                    <div v-if="i != 1" class="absolute top-0 left-0 border-right-1 m-0 p-0 "
                        :style="{ 'height': (size / 2) + 'px', 'width': (size / 2) + 0.782 + 'px' }">
                    </div>
                    <div v-if="j != 9" class="absolute top-0 right-0 border-bottom-1 m-0 p-0"
                        :style="{ 'height': (size / 2) + 'px', 'width': (size / 2) + 'px' }">
                    </div>
                    <div v-if="j != 1" class="absolute bottom-0 left-0 border-top-1 m-0 p-0"
                        :style="{ 'height': (size / 2) + 1 + 'px', 'width': (size / 2) + 'px' }">
                    </div>
                    <div v-if="i != 10" class="absolute bottom-0 right-0 border-left-1 m-0 p-0"
                        :style="{ 'height': (size / 2) + 'px', 'width': (size / 2) + 'px' }">
                    </div>
                    <div v-if="cross_position(j, i)" class="absolute cross"></div>
                    <div v-if="(j == 5 && i == 2) || (j == 5 && i == 9)" class="absolute bigCross" style="height: 50px;">
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const emit = defineEmits(['turn', 'died', 'move'])
const props = defineProps(['size', 'gameSetting', 'enemyMovement'])

const size = ref(props.size)
const sizePixel = size.value * 2 + 'px'

const pieceImages = ref()
onMounted(() => {
    //console.log("mounted")
    getInitialBoard(props.gameSetting.side)
    pieceImages.value = import.meta.glob('../assets/janggi_pieces/*.svg')
})

// 기물 이미지 가져오기
const getPieceImageUrl = (num) => {
    const pieceImagesList = {
        9: `../assets/janggi_pieces/blue_chariot.svg`,
        10: `../assets/janggi_pieces/blue_elephant.svg`,
        11: `../assets/janggi_pieces/blue_horse.svg`,
        12: `../assets/janggi_pieces/blue_cannon.svg`,
        13: `../assets/janggi_pieces/blue_king.svg`,
        14: `../assets/janggi_pieces/blue_advisor.svg`,
        15: `../assets/janggi_pieces/blue_pawn.svg`,
        17: `../assets/janggi_pieces/red_chariot.svg`,
        18: `../assets/janggi_pieces/red_elephant.svg`,
        19: `../assets/janggi_pieces/red_horse.svg`,
        20: `../assets/janggi_pieces/red_cannon.svg`,
        22: `../assets/janggi_pieces/red_advisor.svg`,
        21: `../assets/janggi_pieces/red_king.svg`,
        23: `../assets/janggi_pieces/red_pawn.svg`,
    }
    // https://stackoverflow.com/questions/69696677/vite-vue-3-require-is-not-defined-when-using-image-source-as-props
    // return new URL(pieceImagesList[num], import.meta.url).href;
    return pieceImagesList[num]
}

// 엑스표 위치
const cross_position = (i, j) => {

    const list = [[2, 3], [8, 3], [2, 8], [8, 8], [1, 4], [3, 4], [5, 4], [7, 4], [9, 4], [1, 7], [3, 7], [5, 7], [7, 7], [9, 7]]

    for (let k = 0; k < list.length; k++) {
        if (list[k][0] == i && list[k][1] == j) {
            return true
        }
    }
    return false
}

// 기물 종류
const pieces = ref(
    {
        0: null,
        1: "chariot",
        2: "elephant",
        3: "horse",
        4: "cannon",
        5: "king",
        6: "advisor",
        7: "pawn",
        8: "blue",
        16: "red",
    }
)

// const AddColorsToPiece = (num) => {
//     return
// }

// const getPiece = (num) => {
//     return pieces.value[num]
// }

// 장기판 초기 상태 : 추후에 마상상마, 마상마상, 상마상마, 상마마상 선택 가능하도록 변경
const board = ref(
    [
        [1, 3, 2, 6, 0, 6, 3, 2, 1],
        [0, 0, 0, 0, 5, 0, 0, 0, 0],
        [0, 4, 0, 0, 0, 0, 0, 4, 0],
        [7, 0, 7, 0, 7, 0, 7, 0, 7],
        [0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0],
        [7, 0, 7, 0, 7, 0, 7, 0, 7],
        [0, 4, 0, 0, 0, 0, 0, 4, 0],
        [0, 0, 0, 0, 5, 0, 0, 0, 0],
        [1, 3, 2, 6, 0, 6, 2, 3, 1],
    ]
)

// 상차림 정보가 들어왔을 때, 장기판 정보 변경
const changeBoardPosition = (setting) => {
    for (const key in setting.room.users) {
        const user = setting.room.users[key]
        if (user.Side == setting.side) {
            if (user.formation == '마상상마') {
                board.value[9][1] = 3 //
                board.value[9][2] = 2
                board.value[9][6] = 2
                board.value[9][7] = 3
            } else if (user.formation == '마상마상') {
                board.value[9][1] = 3 //
                board.value[9][2] = 2
                board.value[9][6] = 3
                board.value[9][7] = 2
            } else if (user.formation == '상마상마') {
                board.value[9][1] = 2 //
                board.value[9][2] = 3
                board.value[9][6] = 2
                board.value[9][7] = 3
            } else if (user.formation == '상마마상') {
                board.value[9][1] = 2 //
                board.value[9][2] = 3
                board.value[9][6] = 3
                board.value[9][7] = 2
            }
        } else {
            if (user.formation == '마상상마') {
                board.value[0][7] = 3 //
                board.value[0][6] = 2
                board.value[0][2] = 2
                board.value[0][1] = 3
            } else if (user.formation == '마상마상') {
                board.value[0][7] = 3 //
                board.value[0][6] = 2
                board.value[0][2] = 3
                board.value[0][1] = 2
            } else if (user.formation == '상마상마') {
                board.value[0][7] = 2 //
                board.value[0][6] = 3
                board.value[0][2] = 2
                board.value[0][1] = 3
            } else if (user.formation == '상마마상') {
                board.value[0][7] = 2 //
                board.value[0][6] = 3
                board.value[0][2] = 3
                board.value[0][1] = 2
            }
        }
    }
}

const tempBoard = ref([])

// 왕의 위치, 아군의 왕은 0번째 배열, 적의 왕은 1번째 배열
const kingPosition = ref([
    [8, 4],
    [1, 4]
])

const palacePostionDelta = {
    '[0,3]': [[1, 1]],
    '[0,5]': [[1, -1]],
    '[1,4]': [[1, 1], [1, -1], [-1, 1], [-1, -1]],
    '[2,3]': [[-1, 1]],
    '[2,5]': [[-1, -1]],
    '[7,3]': [[1, 1]],
    '[7,5]': [[1, -1]],
    '[8,4]': [[1, 1], [1, -1], [-1, 1], [-1, -1]],
    '[9,3]': [[-1, 1]],
    '[9,5]': [[-1, -1]]
}

var mySide = null
const side = ref(null)

// 진영 결정에 따른 기물 정보 변경 : 초나라 +8, 한나라 +16
const getInitialBoard = (color) => {
    changeBoardPosition(props.gameSetting)
    if (color == 8) {
        mySide = 8
        side.value = 8
        for (var i in board.value) {
            if (i <= 3) {
                for (var j in board.value[i]) {
                    if (board.value[i][j] != 0) {
                        board.value[i][j] += 16
                    }
                }
            }
            if (i >= 6) {
                for (var j in board.value[i]) {
                    if (board.value[i][j] != 0) {
                        board.value[i][j] += 8
                    }
                }
            }
        }
    }
    else {
        mySide = 16
        side.value = 16
        for (var i in board.value) {
            if (i <= 3) {
                for (var j in board.value[i]) {
                    if (board.value[i][j] != 0) {
                        board.value[i][j] += 8
                    }
                }
            }
            if (i >= 6) {
                for (var j in board.value[i]) {
                    if (board.value[i][j] != 0) {
                        board.value[i][j] += 16
                    }
                }
            }
        }
    }
    // //console.log(board.value)
}

// 선택 된 기물
const seletedPiece = ref(null)

// 좌표 위치의 기물이 선택 되었는지 확인
const isSelected = (i, j) => {
    if (seletedPiece.value == null) {
        return false
    }
    else if (seletedPiece.value[0] == i && seletedPiece.value[1] == j) {
        return true
    }
    else {
        return false
    }
}

// 마지막으로 움직인 위치의 기물
const lastMove = ref(null)

// 좌표 위치의 기물이 마지막 움직인 위치의 기물인지 확인
const isLastMovde = (i, j) => {
    if (lastMove.value == null) {
        return false
    }
    else if (lastMove.value[0] == i && lastMove.value[1] == j) {
        return true
    }
    else {
        return false
    }
}

// 플레이 턴 : 초나라 8, 한나라 16
const turn = ref(8)

// 선택한 위치의 기물이 턴에 맞는지 확인
// 매칭 된 게임에서는, 턴이 넘어가도 적 기물을 클릭할 수 없음.
const isOnTurn = (i, j) => {

    // if (side.value - board.value[i][j] > 8) {
    //     console.log(side.value - board.value[i][j])
    //     // console.log(i, j, board.value[i][j] - side.value)
    //     return true
    // }

    if (turn.value == 8) {
        if (board.value[i][j] < 16 && side.value == turn.value) {
            return false // no 'pointer-events: none;' in css
        }
        else {
            return true
        }
    }
    else {
        if (board.value[i][j] > 16 && side.value == turn.value) {
            return false
        }
        else {
            return true
        }
    }
}

// 선택한 기물의 움직일 수 있는 경로
const availableMoves = ref(null)
const printAvailableMoves = ref(null)

// 선택한 기물의 움직일 수 있는 경로 확인
const isMoveAvailable = (i, j) => {
    if (printAvailableMoves.value == null) {
        return false
    }
    else {
        for (const move in printAvailableMoves.value) {
            if (printAvailableMoves.value[move][0] == i && printAvailableMoves.value[move][1] == j) {
                return true
            }
        }
        return false
    }
}

// 장기판 위의 좌표값을 확인하며 선택한 기물의 경로를 계산 
// const pathFindingOption.value = 
const pathFinding = (i, j, option) => {
    var delta
    availableMoves.value = null

    const num = board.value[i][j]

    var selection
    if (option == undefined) {
        selection = true
    } else {
        selection = option
    }
    if (selection) {
        if (seletedPiece.value != null) {
            if (seletedPiece.value[0] == i && seletedPiece.value[1] == j) {
                seletedPiece.value = null
                availableMoves.value = null
                printAvailableMoves.value = null
                return
            }
        }
        seletedPiece.value = [i, j]
    }

    switch (num) {
        case 9: // 초나라 차
            delta = [[-1, 0], [0, 1], [1, 0], [0, -1]]
            getChariotMovement(i, j, delta)
            break
        case 10: // 초나라 상
            delta = [[-1, 0], [0, 1], [1, 0], [0, -1]]
            getElephantMovement(i, j, delta)
            break
        case 11: // 초나라 마
            delta = [[-1, 0], [0, 1], [1, 0], [0, -1]]
            getHorseMovement(i, j, delta)
            break
        case 12: // 초나라 포
            delta = [[-1, 0], [0, 1], [1, 0], [0, -1]]
            getCannonMovement(i, j, delta)
            break
        case 13: // 초나라 왕
            delta = [[-1, 0], [0, 1], [1, 0], [0, -1]]
            getKingMovement(i, j, delta)
            break
        case 14: // 초나라 사
            delta = [[-1, 0], [0, 1], [1, 0], [0, -1]]
            getAdvisorMovement(i, j, delta)
            break
        case 15: // 초나라 졸
            if (mySide == 8) {
                delta = [[0, -1], [-1, 0], [0, 1]]
            } else {
                delta = [[0, -1], [1, 0], [0, 1]]
            }
            getPawnMovement(i, j, delta)
            break
        case 17: // 한나라 차
            delta = [[-1, 0], [0, 1], [1, 0], [0, -1]]
            getChariotMovement(i, j, delta)
            break
        case 18: // 한나라 상
            delta = [[-1, 0], [0, 1], [1, 0], [0, -1]]
            getElephantMovement(i, j, delta)
            break
        case 19: // 한나라 마
            delta = [[-1, 0], [0, 1], [1, 0], [0, -1]]
            getHorseMovement(i, j, delta)
            break
        case 20: // 한나라 포
            delta = [[-1, 0], [0, 1], [1, 0], [0, -1]]
            getCannonMovement(i, j, delta)
            break
        case 21: // 한나라 왕
            delta = [[-1, 0], [0, 1], [1, 0], [0, -1]]
            getKingMovement(i, j, delta)
            break
        case 22: // 한나라 사
            delta = [[-1, 0], [0, 1], [1, 0], [0, -1]]
            getAdvisorMovement(i, j, delta)
            break
        case 23: // 한나라 병
            if (mySide == 16) {
                delta = [[0, -1], [-1, 0], [0, 1]]
            } else {
                delta = [[0, -1], [1, 0], [0, 1]]
            }
            getPawnMovement(i, j, delta)
            break
        default:
            break
    }

    if (!selection) {
        console.log('in pathfinding :', availableMoves.value)
        console.log(i, j, num)
    }

    if (selection) {
        printAvailableMoves.value = JSON.parse(JSON.stringify(availableMoves.value))
    }

}

const janggun = ref(false)

// 선택한 기물을 선택한 좌표로 이동
const move = (row, column) => {

    // 움직이는 위치에 기물이 있는지 확인
    if (board.value[row][column] != 0) {
        emit('died', board.value[row][column])
    }

    // 장기판 정보 변경
    board.value[row][column] = board.value[seletedPiece.value[0]][seletedPiece.value[1]]

    // 마지막 움직임 지점 저장
    lastMove.value = [row, column]

    const movement = {
        previous: seletedPiece.value,
        current: lastMove.value
    }

    if (turn.value == mySide) {
        emit('move', movement)
    }

    // 위험한 방식일 수도 있지만, 이미 클릭이 가능하다는 전제하에 이렇게 처리함. 
    // => 체크메이트 확인은 다른 곳에서 처리 (watch 등을 이용)
    if (janggun.value) {
        janggun.value = false
        alert("멍군입니다.")
    }

    if (board.value[row][column] % 8 == 5) {
        if (turn.value == mySide) {
            kingPosition.value[0] = [row, column]
        }
        else {
            kingPosition.value[1] = [row, column]
        }
    }

    // 움직이고 난 뒤의 자리는 0으로 변경
    board.value[seletedPiece.value[0]][seletedPiece.value[1]] = 0

    // 차례 변경
    turn.value = turn.value == 8 ? 16 : 8
    emit('turn', turn.value)

    // 장군 확인 : 적의 턴을 가정해서, 적 왕의 안전을 확인
    if (!isKingSafe(seletedPiece.value[0], seletedPiece.value[1], row, column, true)) {
        // check the Checkmate(외통수)
        if (isCheckmate()) {
            alert('게임 종료')
        }
        else {
            janggun.value = true
            alert("장군입니다.")
        }
    }

    // 초기화
    seletedPiece.value = null
    availableMoves.value = null
    printAvailableMoves.value = null
}

// 외통수 확인
const isCheckmate = () => {
    // 해당하는 턴의 아군 기물들을 경로를 모두 조사한 후, 움직일 경로가 없다면 외통수
    // 1. 보드에서 해당 턴의 기물들을 모두 찾는다.
    for (const i in board.value) {
        for (const j in board.value[i]) {
            if (board.value[i][j] != 0 && !isEnemy(board.value[i][j], turn.value, 5)) {
                // 2. 기물의 경로를 조사한다.
                pathFinding(Number(i), Number(j), false, true)
                // 3. 경로가 있다면 외통수가 아니다.
                // console.log(board.value[i][j], Number(i), Number(j), availableMoves.value.length, availableMoves.value)
                if (availableMoves.value.length != 0) {
                    return false
                }
            }
        }
    }
    return true
}


// 졸병의 움직임 계산
const getPawnMovement = (i, j, delta) => {
    availableMoves.value = []

    const key = '[' + i + ',' + j + ']'
    if (palacePostionDelta[key] != undefined) {
        const palacePostion = palacePostionDelta[key]
        // add palacePostion's Deleta value in delta
        for (const p in palacePostion) {
            if (turn.value == mySide && palacePostion[p][0] == 1) {
                continue
            } else if (turn.value != mySide && palacePostion[p][0] == -1) {
                continue
            } else {
                delta.push(palacePostion[p])
            }
        }
    }

    for (const d in delta) {
        const row = i + delta[d][0]
        const column = j + delta[d][1]
        // //console.log(board.value[column][row])
        if (row < 0 || row > 9 || column < 0 || column > 8) {
            continue
        }
        if (board.value[row][column] == 0 || isEnemy(board.value[row][column], board.value[i][j], 5)) {
            // 움직이기 전 왕이 안전한지 확인
            if (isKingSafe(i, j, row, column)) {
                availableMoves.value.push([row, column])
            }
        }
    }
}

// 왕의 움직임 계산
const getKingMovement = (i, j, delta) => {
    movingInPalace(i, j, delta)
}

// 사의 움직임 계산
const getAdvisorMovement = (i, j, delta) => {
    movingInPalace(i, j, delta)
}

// 궁 안에서 움직이는 왕과 사의 기본적인 움직임
const movingInPalace = (i, j, delta) => {
    availableMoves.value = []
    const key = '[' + i + ',' + j + ']'
    const palacePostion = palacePostionDelta[key]

    for (const p in palacePostion) {
        delta.push(palacePostion[p])
    }

    for (const d in delta) {
        const row = i + delta[d][0]
        const column = j + delta[d][1]
        if ((turn.value == mySide) && (row < 7 || row > 9 || column < 3 || column > 5)) {
            continue
        }
        if ((turn.value != mySide) && (row < 0 || row > 2 || column < 3 || column > 5)) {
            continue
        }
        if (board.value[row][column] == 0 || isEnemy(board.value[row][column], board.value[i][j], 5)) {

            if (isKingSafe(i, j, row, column)) {
                availableMoves.value.push([row, column])
                console.log('check', row, column, availableMoves.value)
            }
        }
    }
}


// 마의 움직임 계산
const getHorseMovement = (i, j, delta) => {
    availableMoves.value = []
    for (const d in delta) {
        const row = i + delta[d][0]
        const column = j + delta[d][1]
        if (row < 0 || row > 9 || column < 0 || column > 8) {
            continue
        }
        if (board.value[row][column] == 0) {
            var newDelta = []
            if (delta[d][0] == 0) {
                newDelta = [[1, delta[d][1]], [-1, delta[d][1]]]
            }
            else {
                newDelta = [[delta[d][0], 1], [delta[d][0], -1]]
            }
            // availableMoves.value.push([row, column])
            for (const nd in newDelta) {
                var newRow = row + newDelta[nd][0]
                var newColumn = column + newDelta[nd][1]

                if (newRow < 0 || newRow > 9 || newColumn < 0 || newColumn > 8) {
                    continue
                }
                if (board.value[newRow][newColumn] == 0 || isEnemy(board.value[newRow][newColumn], board.value[i][j], 5)) {
                    if (isKingSafe(i, j, newRow, newColumn)) {
                        availableMoves.value.push([newRow, newColumn])
                    }
                }
            }
        }
    }
}

// 상의 움직임 계산
const getElephantMovement = (i, j, delta) => {
    availableMoves.value = []
    for (const d in delta) {
        const row = i + delta[d][0]
        const column = j + delta[d][1]
        if (row < 0 || row > 9 || column < 0 || column > 8) {
            continue
        }
        if (board.value[row][column] == 0) {
            var newDelta = []
            if (delta[d][0] == 0) {
                newDelta = [[1, delta[d][1]], [-1, delta[d][1]]]
            }
            else {
                newDelta = [[delta[d][0], 1], [delta[d][0], -1]]
            }

            for (const nd in newDelta) {
                const newRow = row + newDelta[nd][0]
                const newColumn = column + newDelta[nd][1]

                if (newRow < 0 || newRow > 9 || newColumn < 0 || newColumn > 8) {
                    continue
                }
                if (board.value[newRow][newColumn] == 0) {
                    for (const ld in newDelta) {
                        const lastRow = newRow + newDelta[nd][0]
                        const lastColumn = newColumn + newDelta[nd][1]

                        if (lastRow < 0 || lastRow > 9 || lastColumn < 0 || lastColumn > 8) {
                            continue
                        }
                        if (board.value[lastRow][lastColumn] == 0 || isEnemy(board.value[lastRow][lastColumn], board.value[i][j], 5)) {
                            if (isKingSafe(i, j, lastRow, lastColumn)) {
                                availableMoves.value.push([lastRow, lastColumn])
                            }
                        }
                    }
                }
            }
        }
    }
}

// 차의 움직임 계산
const getChariotMovement = (i, j, delta) => {
    availableMoves.value = []
    var initial = board.value[i][j]

    // 위치가 궁 안 인지 확인한 후에 dt 값을 시작지점에 따라 변경.
    // 추가된 dt 값은 궁 안에서만 움직일 수 있도록 함.
    const key = '[' + i + ',' + j + ']'
    if (palacePostionDelta[key] != undefined) {
        const palacePostion = palacePostionDelta[key]
        // add palacePostion's Deleta value in delta
        for (const p in palacePostion) {
            delta.push(palacePostion[p])
        }
    }


    for (const d in delta) {
        forwarding(i, j, delta[d], initial)
    }
}

// 전진 확인 위한 재귀 함수
const forwarding = (i, j, delta, initial) => {

    const row = i + delta[0]
    const column = j + delta[1]

    if (row < 0 || row > 9 || column < 0 || column > 8) {
        return
    }

    // 델타 값이 x, y 축 모두 0이 아니라면 이는 궁 안에서 가질 수 있는 움직임이다. 
    // 만약 다음 경로 값이 궁 밖을 벗어난다면, 이 경로와 그 이후는 고려하지 않는다.
    if ((delta[0] != 0 && delta[1] != 0) && (row < 0 || row > 2 || column < 3 || column > 5) && (row < 7 || row > 9 || column < 3 || column > 5)) {
        return
    }

    if (board.value[row][column] == 0) {
        if (isKingSafe(i, j, row, column)) {
            availableMoves.value.push([row, column])
        }
        forwarding(row, column, delta, initial)
    }
    else if (isEnemy(board.value[row][column], initial, 5)) {
        if (isKingSafe(i, j, row, column)) {
            availableMoves.value.push([row, column])
        }
    }
    return
}

// 포의 움직임 계산
const getCannonMovement = (i, j, delta) => {
    availableMoves.value = []
    var initial = [board.value[i][j], i, j]

    const key = '[' + i + ',' + j + ']'
    if (palacePostionDelta[key] != undefined) {
        const palacePostion = palacePostionDelta[key]
        for (const p in palacePostion) {
            delta.push(palacePostion[p])
        }
    }

    for (const d in delta) {
        beforeJump.value = []
        const check = checkBlock(i, j, delta[d])
        if (check) {
            jumping(beforeJump.value[0], beforeJump.value[1], delta[d], initial)
        }
    }
}

// 
const beforeJump = ref([])

// 포의 점프 가능 여부 확인 : 막는 경로가 있으면 점프 가능
const checkBlock = (i, j, delta) => {
    const row = i + delta[0]
    const column = j + delta[1]
    if (row < 0 || row > 9 || column < 0 || column > 8) {
        return false
    }

    if ((delta[0] != 0 && delta[1] != 0) && (row < 0 || row > 2 || column < 3 || column > 5) && (row < 7 || row > 9 || column < 3 || column > 5)) {
        return false
    }

    if (board.value[row][column] == 0) {
        const check = checkBlock(row, column, delta)
        if (check) {
            return true
        }
        return false
    }
    else if (board.value[row][column] != 0 && board.value[row][column] != 12 && board.value[row][column] != 20) {
        beforeJump.value = [row, column]
        return true
    }
}

// 포의 점프 확인 위한 재귀 함수
const jumping = (i, j, delta, initial) => {

    const row = i + delta[0]
    const column = j + delta[1]

    if (row < 0 || row > 9 || column < 0 || column > 8) {
        return
    }
    if ((delta[0] != 0 && delta[1] != 0) && (row < 0 || row > 2 || column < 3 || column > 5) && (row < 7 || row > 9 || column < 3 || column > 5)) {
        return
    }

    if (board.value[row][column] == 0) {
        console.log(row, column)
        if (isKingSafe(initial[1], initial[2], row, column)) {
            availableMoves.value.push([row, column])
        }
        jumping(row, column, delta, initial)
    }
    else if (isEnemy(board.value[row][column], initial[0], 5) && board.value[row][column] != 12 && board.value[row][column] != 20) {
        if (isKingSafe(initial[1], initial[2], row, column)) {
            availableMoves.value.push([row, column])
        }
    }
    return
}

// 피아 식별을 위한 계산 함수 - 숫자의 2진수 앞 자리를 가지고 계산
// 적이면 true, 아군이면 false
function isEnemy(num1, num2, position) {
    // Convert numbers to binary strings with leading zeros
    const binary1 = num1.toString(2).padStart(position, '0');
    const binary2 = num2.toString(2).padStart(position, '0');
    // Check if the bits at the specified position are the same
    return binary1[0] !== binary2[0];
}


// 왕의 위치를 중심으로, 차, 포, 마, 상, 졸병의 움직임을 역으로 계산하여 왕이 잡히는지 확인
// 잡히면 true, 아니면 false
// 아군 차례일 때는 기물 움직임 전에 이 함수가 호출됨
// 적의 차례일 때는 기물 움직임 이후에 이 함수가 호출됨
const isKingSafe = (i_origin, j_origin, row, column, check) => {

    if (check == undefined) {
        check = false
    }

    if (check) {
        var tempBoard = JSON.parse(JSON.stringify(board.value))
    } else {
        var tempBoard = JSON.parse(JSON.stringify(board.value))
        tempBoard[row][column] = board.value[i_origin][j_origin]
        tempBoard[i_origin][j_origin] = 0
    }

    if (turn.value == mySide) {
        var king
        if (board.value[i_origin][j_origin] % 8 == 5) {
            king = [row, column]
        } else {
            king = kingPosition.value[0]
        }
        const i = king[0]
        const j = king[1]

        const key = '[' + i + ',' + j + ']'
        if (palacePostionDelta[key] != undefined) {
            var delta = palacePostionDelta[key]
        }
        return reversePathFinding(i, j, delta, tempBoard)
    }
    else {
        if (board.value[i_origin][j_origin] % 8 == 5) {
            king = [row, column]
        } else {
            king = kingPosition.value[1]
        }
        const i = king[0]
        const j = king[1]

        const key = '[' + i + ',' + j + ']'
        if (palacePostionDelta[key] != undefined) {
            var delta = palacePostionDelta[key]
        }
        return reversePathFinding(i, j, delta, tempBoard)
    }
}


const reverseBeforeJump = ref([])
const reversePathFinding = (i, j, additionalDelta, tempBoard) => {
    // 1. 차포졸병의 위협 확인
    var delta1 = [[-1, 0], [0, 1], [1, 0], [0, -1]]
    for (const p in additionalDelta) {
        delta1.push(additionalDelta[p])
    }
    reverseBeforeJump.value = []
    for (const d in delta1) {
        if (!reversePawn(i, j, delta1[d], tempBoard)) {
            // console.log("king is in danger by pawn", reversePawn(i, j, delta1[d], tempBoard))
            return false
        }

        if (!reverseChariot(i, j, delta1[d], tempBoard)) {
            // console.log("king is in danger by chariot")
            return false
        }
    }

    if (!reverseCanonn(tempBoard)) {
        // console.log("king is in danger by cannon")
        return false
    }

    // 2. 마상의 위협확인
    var delta2 = [[1, 1], [-1, 1], [1, -1], [-1, -1]]
    for (const d in delta2) {
        if (!reverseHorseAndElephant(i, j, delta2[d], tempBoard)) {
            // console.log("king is in danger by horse or elephant")
            return false
        }
    }

    return true
}

const reversePawn = (i, j, delta, tempBoard) => {
    const row = i + delta[0]
    const column = j + delta[1]

    if (row < 0 || row > 9 || column < 0 || column > 8) {
        return true
    }
    if ((delta[0] != 0 && delta[1] != 0) && (row < 0 || row > 2 || column < 3 || column > 5) && (row < 7 || row > 9 || column < 3 || column > 5)) {
        return true
    }
    if (tempBoard[row][column] == 0) {
        return true
    } else if (isEnemy(tempBoard[row][column], 5 + turn.value, 5)) {
        const piece = tempBoard[row][column] % 8
        //아랫쪽 궁 기준 delta 값이 [1,0], [1, -1], [1, 1]이 아닐 때, 적 졸병이 있으면 위험한 상황
        if (turn.value == 8 && piece == 7 && delta[0] != 1) {
            // console.log("danger!!! threat by pawn")
            return false
        } else if (turn.value == 16 && piece == 7 && delta[0] != -1) {
            // console.log("danger!!! threat by pawn")
            return false
        }
        return true
    }
    return true
}

const reverseChariot = (i, j, delta, tempBoard) => {
    const row = i + delta[0]
    const column = j + delta[1]

    if (row < 0 || row > 9 || column < 0 || column > 8) {
        return true
    }

    if ((delta[0] != 0 && delta[1] != 0) && (row < 0 || row > 2 || column < 3 || column > 5) && (row < 7 || row > 9 || column < 3 || column > 5)) {
        return true
    }

    if (tempBoard[row][column] != 0 && tempBoard[row][column] != 12 && tempBoard[row][column] != 20) {
        reverseBeforeJump.value.push([row, column, delta])
    }
    if (tempBoard[row][column] == 0) {
        if (!reverseChariot(row, column, delta, tempBoard)) {
            return false
        }
        return true
    } else if (isEnemy(tempBoard[row][column], 5 + turn.value, 5)) {
        const piece = tempBoard[row][column] % 8
        // 경로 상에 상대방 차가 있으면 위험한 상황
        if (piece == 1) {
            // console.log("danger!!! threat by chariot")
            return false
        }
        return true
    }
    return true
}

const reverseCanonn = (tempBoard) => {
    for (const jump in reverseBeforeJump.value) {
        if (!reverseJumping(reverseBeforeJump.value[jump][0], reverseBeforeJump.value[jump][1], reverseBeforeJump.value[jump][2], tempBoard)) {
            return false
        }
    }
    return true
}

const reverseJumping = (i, j, delta, tempBoard) => {
    const row = i + delta[0]
    const column = j + delta[1]

    if (row < 0 || row > 9 || column < 0 || column > 8) {
        return true
    }
    if ((delta[0] != 0 && delta[1] != 0) && (row < 0 || row > 2 || column < 3 || column > 5) && (row < 7 || row > 9 || column < 3 || column > 5)) {
        return true
    }

    if (tempBoard[row][column] == 0) {
        if (reverseJumping(row, column, delta, tempBoard)) {
            return true
        }
        return false
    }
    else if (isEnemy(tempBoard[row][column], 5 + turn.value, 5)) {
        const piece = tempBoard[row][column] % 8
        if (piece == 4) {
            console.log("danger!!! threat by cannon")
            return false
        }
    }
    return true
}

const reverseHorseAndElephant = (i, j, delta, tempBoard) => {
    var row = i + delta[0]
    var column = j + delta[1]
    if (row < 0 || row > 9 || column < 0 || column > 8) {
        return true
    }
    // 첫 번째 대각선
    if (tempBoard[row][column] == 0) {

        // check if Horse existed
        var newRow = row + delta[0]
        var newColumn = column + 0
        if (newRow < 0 || newRow > 9 || newColumn < 0 || newColumn > 8) {
            return true
        }
        if (isEnemy(tempBoard[newRow][newColumn], 5 + turn.value, 5)) {
            var piece = tempBoard[newRow][newColumn] % 8
            if (piece == 3) {
                return false
            }
        }

        newRow = row + 0
        newColumn = column + delta[0]

        if (newRow < 0 || newRow > 9 || newColumn < 0 || newColumn > 8) {
            return true
        }
        if (isEnemy(tempBoard[newRow][newColumn], 5 + turn.value, 5)) {
            piece = tempBoard[newRow][newColumn] % 8
            if (piece == 3) {
                return false
            }
        }

        // check if elephant existed

        row = row + delta[0]
        column = column + delta[1]
        if (row < 0 || row > 9 || column < 0 || column > 8) {
            return true
        }

        if (tempBoard[row][column] == 0) {
            // check if Elephant existed
            newRow = row + delta[0]
            newColumn = column + 0
            if (newRow < 0 || newRow > 9 || newColumn < 0 || newColumn > 8) {
                return true
            }
            if (isEnemy(tempBoard[newRow][newColumn], 5 + turn.value, 5)) {
                piece = tempBoard[newRow][newColumn] % 8
                if (piece == 2) {
                    return false
                }
            }

            newRow = row + 0
            newColumn = column + delta[0]

            if (newRow < 0 || newRow > 9 || newColumn < 0 || newColumn > 8) {
                return true
            }
            if (isEnemy(tempBoard[newRow][newColumn], 5 + turn.value, 5)) {
                piece = tempBoard[newRow][newColumn] % 8
                if (piece == 2) {
                    return false
                }
            }
        }
    }
    return true
}

defineExpose({
    seletedPiece,
    move,
})

</script>

<style scoped>
.not_draggable {
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -o-user-select: none;
    user-select: none;
    -webkit-user-drag: none;
    -khtml-user-drag: none;
    -moz-user-drag: none;
    -o-user-drag: none;
    user-drag: none;
}


.diagonalLine {
    position: absolute;
    top: 50%;
    left: 50%;
    border-top: 1px solid black;
    width: 10%;
    height: 10%;
    transform: rotate(90deg);
    transform-origin: 0 0;
}

.cross {
    position: relative;
    top: 50%;
    left: 50%;
}

.cross:before,
.cross:after {
    position: absolute;
    right: 0%;
    top: -6px;
    content: '';
    height: 12px;
    width: 1px;
    background-color: black;
}

.cross:before {
    transform: rotate(45deg);
}

.cross:after {
    transform: rotate(-45deg);
}


.bigCross {
    position: relative;
    top: 50%;
    left: 50%;
}

.bigCross:before,
.bigCross:after {
    --sqrt2: 1.4142135623730951;
    --size: v-bind(sizePixel);
    position: absolute;
    right: 0%;
    top: calc(var(--size) * -0.5 * var(--sqrt2));
    content: '';
    height: calc(var(--size) * var(--sqrt2));
    width: 1px;
    background-color: black;
}

.bigCross:before {
    transform: rotate(45deg);
}

.bigCross:after {
    transform: rotate(-45deg);
}
</style>
